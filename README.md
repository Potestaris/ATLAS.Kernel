# DEPRECATED
# KUKULCAN.Kernel

KUKULCAN.Kernel is the cross-cutting core of the "Kukulcán Software Design" application ecosystem.
It defines the domain primitives, infrastructure contracts, extensions, and MediatR pipeline behaviors used by each module, as well as the immutable components that ensure consistency, interoperability, and stability across all defined contexts and services.

This repository acts as the single source of truth for shared concepts, preventing duplication, semantic drift, and circular dependencies between modules.

## Purpose

- Unify the ubiquitous language across the ATLAS platform.
- Provide stable, reusable building blocks for all modules.
- Maintain semantic consistency between bounded contexts.
- Reduce duplication of code and domain concepts.
- Establish clear, versioned contracts for internal communication.
- Serve as the foundation for controlled domain evolution.

## Projects

| Project                         | Description                                                           | Dependencies                     |
|---------------------------------|-----------------------------------------------------------------------|----------------------------------|
| `KUKULCAN.Kernel.Abstractions`    | Pure interfaces and contracts — zero implementation dependencies       | None                             |
| `KUKULCAN.Kernel.Domain`          | Entities, Value Objects, Events, Result pattern, Specifications, Guards | Abstractions                     |
| `KUKULCAN.Kernel.Infrastructure`  | Extensions, Pagination, SequentialGuid, MediatR Behaviors              | Domain + MediatR + FluentValidation |

## Entity Hierarchy

```
IEntity<TId>
└── EntityBase<TId>                         // Id + structural equality
      └── AuditableEntityBase<TId>          // + CreatedAt/By, UpdatedAt/By
            ├── TenantEntityBase<TId>       // + TenantId + ISoftDeletable  ← operational data
            │     └── AggregateRoot<TId>    // + Domain Events              ← aggregate roots
            ├── MasterEntity<TId>           // IMasterData + IActivatable   ← Countries, Currencies
            ├── ReferenceEntity<TId>        // IMasterData + IActivatable   ← InvoiceStatus (global)
            └── TenantReferenceEntity<TId>  // IMasterData + ITenantAware   ← CustomerStatus (per-tenant)
```

### Deletion Rules

| Base class | Soft-deletable | Tenant-scoped | Notes |
|---|---|---|---|
| `TenantEntityBase<TId>` | ✅ Yes | ✅ Yes | Normal operational entities |
| `AggregateRoot<TId>` | ✅ Yes | ✅ Yes | Aggregate roots |
| `MasterEntity<TId>` | ❌ Never | ❌ No | Global master data (Countries, Currencies) |
| `ReferenceEntity<TId>` | ❌ Never | ❌ No | Global reference values (InvoiceStatus) |
| `TenantReferenceEntity<TId>` | ❌ Never | ✅ Yes | Per-tenant config (CustomerStatus) |

Entities marked with `IImmutable` additionally reject UPDATE and DELETE at infrastructure level.

## MediatR Pipeline (recommended order per module)

```
LoggingBehavior<,>        // 1st — wraps everything, measures total time
TenantBehavior<,>         // 2nd — rejects requests without a resolved tenant
ValidationBehavior<,>     // 3rd — runs FluentValidation, short-circuits on failure
CachingBehavior<,>        // 4th — cache hit/miss for ICacheableRequest queries
TransactionBehavior<,>    // 5th — wraps ITransactionalCommand in a DB transaction (in KUKUKCAN.Database)
```

## Usage Examples

```csharp
// Sequential GUID for SQL Server:
var id = SequentialGuid.NewSequentialGuid();

// Railway pattern:
var result = await mediator.Send(new CreateCustomerCommand(...), ct);
if (result.IsFailure)
    return BadRequest(result.Error);

// Paginated query:
var customers = await mediator.Send(
    new GetCustomerListQuery(PaginationRequest.Create(page: 1, pageSize: 25)), ct);

// Specification:
var spec = new ActiveCustomerSpec().And(new CustomerBySegmentSpec(segmentId: 3));
var items = await dbContext.Customers.ApplySpecification(spec).ToListAsync(ct);
```

## Recommended Folder Structure

```
KUKULCAN.Kernel/
│
├── Documentation/
├── Source/
│   |
│   └── KUKULCAN.Kernel/
│       ├──Extensions/
│       ├──Primitives/
│       |  └──Interfaces/
│       ├── Abstracttions/
│       │   └──Interfaces/
│       │      ├──Domain/
│       |      └──Infrastructure/
│       ├── Domain/
│       │   ├──Entities/
│       │   ├──Events/
│       │   ├──Guards/
│       │   │  ├──Internal/
│       │   ├──Result/
│       │   ├──Specifications/
│       │   └──ValueObjects/
│       └── Infratructure/
│           ├──Behaviors/
│           ├──Extensions/
│           ├──Pagination/
│           └──Primitives/
└── Tests/
    └── KUKUKCAN.Kernel.Tests/
        ├── Extensions/
        ├── Primitives/
        |   └── Interfaces
        ├── Infrastructure/
        ├── TestData/
        ├── Abstractions/
        │   └──Interfaces/
        │      ├──Domain/
        │      └──Infrastructure/
        ├── Domain/
        │   ├──Entities/
        │   ├──Events/
        │   ├──Guards/
        │   │  ├──Internal/
        │   ├──Result/
        │   ├──Specifications/
        │   └──ValueObjects/
        └── Infratructure/
            ├──Behaviors/
            ├──Extensions/
            ├──Pagination/
            └──Primitives/
```

## Testing

This repository includes unit tests for:

- Value Objects  
- Result/Maybe patterns  
- Domain events  
- Exceptions  
- Cross-cutting utilities  

Tests follow AAA (Arrange–Act–Assert), FluentAssertions, and parameterized test patterns.

## Versioning

KUKULCAN.Kernel uses Semantic Versioning (SemVer):

- MAJOR — breaking changes  
- MINOR — backward-compatible features  
- PATCH — backward-compatible fixes  

All changes must go through the internal RFC process.

## Contribution Guidelines

1. Create a branch from `develop`  
2. Follow commit conventions  
3. Add unit tests for new components  
4. Open a Pull Request with a clear description  
5. Await architectural review  

## Requirements

- **.NET 10**
- MediatR 13.*
- FluentValidation 12.*
- Microsoft.Extensions.Logging.Abstractions 10.*

## License

This project is owned and maintained by **Kukulcán Software Designer** and is distributed under the **General Public License (GPL)**.

This means the software is free to use, modify, and redistribute, provided that:

- The original copyright notice is preserved.
- Proper attribution is given to the original creators.
- Any derivative work distributed must remain under the same GPL license terms.

For full license terms, see the `LICENSE` file included in this repository.

## Notes

This library is currently in an early stage and may evolve as the Atlas ecosystem grows.
