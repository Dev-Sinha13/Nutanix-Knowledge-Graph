---
tags: [eng-932537, architecture, cross-repo]
---

# Repository Map & Cross-Repo Flow

Four repos participate. Each owns a distinct layer of the stack; changes must land in this order (top-down by API surface, bottom-up by dependency).

## Per-repo ownership

| Repo | What it owns for `products` | PR |
|---|---|---|
| `ntnx-api-iam` | v4 Swagger / EDM schema definitions for `Entity.products` and `Role.products`. Generates the Go DTOs vendored into iam-themis. (AP schema removed per D24.) | #914 |
| `iam-utils` | `configutil` types (`ConfigObject.ProductList`, `ConfigRole.ProductList`, `ConfigACP.ProductList`), `ProductAllowedTypesMap`, `isConfigValid` validation at config-load time. `themisutil/generated/models/` v1 request+response models (`LoadObjectRequest.Products`, `ClientObject.Products`). (`ConfigACP.ProductList` is unused in iam-themis runtime per D24; harmless.) | #358 + future #TBD (ClientObject swagger) |
| `iam-themis` | Storage layer (column lists, scanners, writers), DB migration, handlers (derivation chokepoints), response builders, OData filter+select wiring, unit tests, tavern E2E. Object + Role only. | #1603 |
| `iam-bootstrap` | Seeds entity `products` from `objects.json` via both v1 (`LoadObjectRequest`) and v4 (`EntityConfig`) bootstrap paths. The historical gap (D-evidence in `iam-bootstrap/.notes/`): both paths dropped `ProductList` silently; fixed in this initiative. | #768 |

## File-level ownership inside iam-themis

| Layer | Files |
|---|---|
| Storage struct | `services/server/storage/storage.go` (`Object.ProductList`, `Role.ProductList`) |
| Column lists + field maps | `services/server/storage/util.go` (`ObjectAllColumns*`, `RoleAllColumns*`; `ObjectFieldsToStorageFields`, `v4RoleFieldsToStorageFields`, etc.; `ObjectSupportedFilterFields`, `RoleSupportedFilterFields`) |
| SQL scan/write | `services/server/storage/sql/client.go` (scanner cases via `pq.Array`); `services/server/storage/sql/role.go`, `object.go` (insert/update statements) |
| Migration | `services/server/storage/sql/migrate.go` (single ENG-932537 block, additive idempotent; object + role tables only) |
| SQLite test schema | `services/server/storage/sql/client.go::beforeTestQueries` |
| Handler derivation | `services/server/apihandler/util.go::ComputeAccessibleEntitiesList` + `getUIDisplayNamesAndProductsForEntities` (role union) |
| Converters | `services/server/storage/util.go::FromLoadObjectRequest` (v1/proxy entity); `FromV4EntityRequest` (v4 entity) |
| Response builders | `services/server/storage/util.go::ToGetObjectResponse` + `ToListObjectResponse` (v1); `services/server/apiutil/object_util.go::ToGetV4ObjectResponse` (v4 entity); `services/server/apiutil/role_util.go::ToGetV4RoleResponse` (v4 role) |
| Vendor (TEMP HACK) | `vendor/github.com/nutanix-core/ntnx-api-iam/iam-server-codegen/models/iam/v4/authz/authz_model.go` (Entity, Role, EntityProjection, RoleProjection — each with `ProductList []string`); `vendor/github.com/nutanix-core/iam-utils/themisutil/generated/models/client_object.go` (`Products []string`) |
| Unit tests | `storage/util_test.go`, `apihandler/util_test.go`, `apiutil/api_util_test.go` (object + role only) |
| Tavern E2E | `api_tests/roles_v4/test_role_product_list_collation.tavern.yaml`, `api_tests/objects/test_object_products_v1_and_proxy.tavern.yaml` |

## File-level ownership inside iam-bootstrap

| Path | What it does |
|---|---|
| `services/themis-bootstrap/configload/configload.go::loadObjects` | v1 seeding path — converts `ConfigObject` to `LoadObjectRequest`. Must carry `ProductList: c.ProductList` (was dropping it silently before this initiative). |
| `services/themis-bootstrap/util/v4_util.go::SeedEntitiesV4` | v4 seeding path — converts `ConfigObject` to `EntityConfig`. Must carry `entity.ProductList = ent.ProductList` (was dropping it silently before this initiative). |
| `services/themis-bootstrap/util/configloadutils.go` | Validates against `ProductAllowedTypesMap` at config-load time. |

## Cross-repo dependency direction

```
ntnx-api-iam (#914)        ← schema source of truth
    ↓ generates Go DTOs
iam-utils (#358 + new)     ← vendors + validates + provides shared types
    ↓ imported by
iam-themis (#1603)         ← storage, handlers, API surface
    ↑ imports
iam-bootstrap (#768)       ← seeds entities at startup, reads same iam-utils types
```

## Data flow: entity `products` end-to-end

For entities, the path from declaration to API response. Roles + APs derive from this and have their own derivation flows in [[06 - Handler Logic]].

```
  objects.json (in iam-themis/services/config/templates/objects/)
      "productList": ["NC"]
        |
        v
  iam-utils/configutil/ConfigObject.ProductList []string  (parsed at startup)
        |
        | iam-bootstrap reads, validates against ProductAllowedTypesMap
        v
  iam-bootstrap chooses v1 or v4 path:
        |
        +--- v1 path: builds models.LoadObjectRequest with Products []string field
        |       POST /api/iam/authz/v1/client/load-objects
        |       iam-themis: serveObjectPOST -> LoadObjectsData -> generateValidatedObjectMap
        |       (Validate(formats) skips ContextValidate so ReadOnly is bypassed)
        |       -> FromLoadObjectRequest copies req.Products -> storage.Object.ProductList
        |
        +--- v4 path: builds v4ResponseModel.EntityConfig with ProductList []string field
                POST /api/iam/v4.0/authz/entities (the seeding endpoint)
                iam-themis: SeedV4Entities -> FromV4EntityRequest copies ent.ProductList -> storage.Object.ProductList
        |
        v
  iam-themis storage:
        SeedObjects -> SQL INSERT/UPDATE with pq.Array(o.ProductList)
        product_list text[] DEFAULT '{}' column persisted
        |
        v
  Reads:
        v1 single GET   /api/iam/authz/v1/objects/{id}        -> ToGetObjectResponse  -> ClientObject.Products
        v1 LIST        /api/iam/authz/v1/objects               -> ToListObjectResponse -> ClientObject.Products
        v4 single GET   /api/iam/v4.0/authz/entities/{extId}    -> ToGetV4ObjectResponse -> Entity.products
        v4 LIST        /api/iam/v4.0/authz/entities             -> Entity.products on each
        OData filter:  ?$filter=products/any(p: p eq 'NC')      -> SQL WHERE product_list && ARRAY['NC']
        OData select:  ?$select=products,displayName            -> projects only those fields
```

For /proxy:

```
  Client POST /api/iam/authz/v1/proxy
  {
    "metadata": { "serviceName": "..." },
    "method": "post",
    "endpoint": "/api/iam/authz/v1/client/load-objects",
    "payload": [ { ..., "products": ["PC"], ... } ]
  }
      |
      v
  iam-themis apihandler/object_seeding.go::loadObjects unmarshals payload to []LoadObjectRequest
      |
      v
  Same LoadObjectsData chokepoint as the v1 direct path — wired identically.
```

## See also

- [[02 - Canonical Spec]] for the field contract this flow propagates
- [[06 - Handler Logic]] for role + AP derivation (which extend this flow)
- [[07 - API Surface Matrix]] for which surfaces are wired today
- Up: [[00 - Index]]
