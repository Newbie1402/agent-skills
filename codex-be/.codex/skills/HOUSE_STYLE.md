# EVC House Style

This file captures the implementation style already present in this repo so future code matches it.

Source patterns were taken from controllers and services such as:
- `AccountingTicketsController`
- `AccountingImportSessionController`
- `CustomerCreditGuaranteeController`
- `InvoiceController`
- `AccountingTicketService` and `AccountingTicketServiceImpl`
- `InvoiceService` and `InvoiceServiceImpl`
- `ExcelImportFacade` and `ExcelImportFacadeImpl`

## Controller style

### Default structure
- Use `@RestController`, `@RequestMapping`, and `@RequiredArgsConstructor`.
- Add `@Tag` and `@Operation` when the surrounding area already uses Swagger annotations.
- Many internal APIs also use `@IsInternal`.
- Some accounting controllers use `@SneakyThrows`; keep the local style of that module instead of forcing it everywhere.

### What controllers should do
- Receive request DTOs and path/query params.
- Apply `@Valid` and custom validation annotations at the edge.
- Call exactly the service needed.
- Optionally map entity or domain result to response DTO if that controller already follows mapper-at-controller style.
- Return via `ResponseEntityGenerator`.

### What controllers should not do
- No accounting formulas.
- No provider-routing logic.
- No repository access.
- No complex branching that belongs in service/helper classes.

### Response pattern
- Single object: `ResponseEntity<ResponseVO<T>>`
- Search/page: `ResponseEntity<ResponsePagingVO<T>>`
- Common wrappers:
  - `okFormat`
  - `createdFormat`
  - `updateFormat`
  - `searchFormat`
  - `findOneFormat`

### Typical controller template

```java
@RestController
@RequestMapping("/api/v1/example")
@RequiredArgsConstructor
@Tag(name = "Example Controller")
public class ExampleController {

    private final ExampleService exampleService;
    private final ExampleMapper exampleMapper;

    @PostMapping
    @IsInternal
    @Operation(summary = "Create example")
    public ResponseEntity<ResponseVO<ExampleResponse>> create(
            @RequestBody @Valid ExampleRequest request) {

        ExampleResponse response =
                exampleMapper.toResponse(exampleService.createExample(request));
        return ResponseEntityGenerator.createdFormat(response);
    }

    @PostMapping("/search")
    @IsInternal
    public ResponseEntity<ResponsePagingVO<ExampleResponse>> search(
            @RequestBody SearchDataDto searchDataDto) {
        Page<ExampleResponse> page = exampleService.searchExamples(searchDataDto);
        return ResponseEntityGenerator.searchFormat(page, searchDataDto);
    }
}
```

### Controller notes from this repo
- If an existing controller already injects a mapper, keep that pattern there.
- If the service already returns DTOs directly, do not force an extra mapping layer in controller.
- Path naming in this repo is practical and endpoint-oriented, not hyper-purist REST. Match nearby endpoints first.

## Service style

### Default structure
- For domain services and facades, the repo often uses interface plus `impl`.
- Use `@Service` and `@RequiredArgsConstructor` on implementations.
- Add `@Slf4j` only when logs are actually needed by that module.
- Helper/domain-specific strategy classes may be concrete services without interface if they are narrow in scope.

### Transaction style
- Query methods: prefer `@Transactional(readOnly = true)` when the service already follows that pattern.
- Write methods: use `@Transactional`.
- Keep transaction boundaries in service/facade layer, not controller.

### What services should do
- Orchestrate repositories, clients, mappers, converters, and helper services.
- Own business decisions such as provider routing, import strategy, status changes, and accounting formulas.
- Throw project exceptions like `BaseError` with existing enum/error definitions.
- Hide repeated rule fragments in private methods or dedicated helper services.

### What services should not do
- Do not return raw Feign details to controller unless the current module already does that.
- Do not leak repository logic into controller.
- Do not duplicate the same formula in manual create flow and import flow.

### Naming patterns seen in repo
- `createX`
- `updateX`
- `getXById`
- `searchX`
- `findByX`
- `commitSession`
- `resolveConflict`
- `initiateImport`

### Service template

```java
public interface ExampleService {

    Page<ExampleResponse> searchExamples(SearchDataDto searchDataDto);

    ExampleEntity createExample(ExampleRequest request);
}
```

```java
@Service
@RequiredArgsConstructor
public class ExampleServiceImpl implements ExampleService {

    private final ExampleRepository exampleRepository;
    private final ExampleMapper exampleMapper;
    private final ExternalClient externalClient;

    @Override
    @Transactional(readOnly = true)
    public Page<ExampleResponse> searchExamples(SearchDataDto searchDataDto) {
        Pageable pageable = SearchUtil.getPageable(searchDataDto);
        Specification<ExampleEntity> spec =
                SearchUtil.getSpecification(searchDataDto, ExampleEntity.class);
        return exampleRepository.findAll(spec, pageable).map(exampleMapper::toResponse);
    }

    @Override
    @Transactional
    public ExampleEntity createExample(ExampleRequest request) {
        validateBusinessRule(request);

        ExampleEntity entity = exampleMapper.toEntity(request);

        if (shouldCallExternal(request)) {
            ExternalResult result = externalClient.call(...);
            applyExternalResult(entity, result);
        }

        return exampleRepository.save(entity);
    }

    private void validateBusinessRule(ExampleRequest request) {
        // Keep business validation in service.
    }
}
```

## Patterns to preserve

### Search endpoints
- Search requests commonly accept `SearchDataDto`.
- Pageable and `Specification` are often built with `SearchUtil`.
- Page responses go through `ResponseEntityGenerator.searchFormat(...)`.

### Accounting and import flows
- Manual create flow and Excel import flow are separate entry points, but business parity matters.
- Complex workflows are often represented as facade plus helper services.
- Import/session flows keep parsing, staging, reconciliation, and commit responsibilities separated.

### Validation and errors
- Bean validation stays on DTOs and controller params.
- Business validation stays in service.
- Reuse existing error enums and `BaseError` instead of inventing ad hoc exception strings.

### Method parameters

- Avoid long parameter lists.
- Methods should generally have no more than 3 parameters.
- When an operation requires more than 3 related parameters, group them into a dedicated Request DTO, Command, Query, or parameter object according to the layer and purpose.
- Parameters that together represent one business concept may be grouped even when there are only 3.
- Do not introduce wrapper objects solely to reduce the parameter count when doing so would make the code less clear.

## Mapper style

### Local module conventions

- Follow the mapping style already used by the surrounding module.
- Some modules use dedicated mapper classes (for example, accounting flows).
- Some modules assemble DTOs directly inside services (for example, parts of the invoice flow).
- Do not introduce a different mapping style into an existing module unless explicitly requested.

### Responsibility

- A mapper converts one object graph into another object graph.
- Prefer MapStruct when the surrounding module already uses MapStruct.
- Typical mappings:
  - request DTO -> entity
  - entity -> response DTO
  - external/provider model -> internal model
  - page/list element mapping
- Mapper methods must be deterministic and side-effect free.

### Allowed in mappers

- Copying and renaming fields.
- Flattening or nesting object structures.
- Null-safe structural conversion.
- Simple representation conversion required by the target model.
- Calling dedicated converters through MapStruct `uses`.
- Applying mapping constants that are part of the representation contract.

### Not allowed in mappers

- Repository, Redis, Feign client, event publisher, or service calls.
- Database lookups.
- Business validation or permission checks.
- Provider-routing decisions.
- Accounting formulas.
- Status-transition decisions.
- Generating values whose meaning depends on the current business operation.
- Mutating unrelated existing entities.

### Update mapping

- Use `@MappingTarget` when updating an existing entity.
- Ignore immutable/system-managed fields such as id, createdAt, createdBy, and version unless the local module explicitly manages them through mapping.
- Configure null handling explicitly when partial updates are supported.
- Do not silently overwrite existing fields with null unless that is the endpoint contract.

### Placement

- Mapper interfaces/classes belong near the owning domain module.
- Reuse an existing mapper when the source and target semantics are the same.
- Do not create a generic mapper abstraction only to remove a small amount of duplication.

### Naming

- `toEntity`
- `toResponse`
- `toResponses`
- `updateEntity`
- Use more explicit names when multiple source models exist, such as:
  - `providerTicketToEntity`
  - `importRowToStagingEntity`
  
## Converter style

### Responsibility

- A converter transforms one value or one narrowly scoped representation into another.
- Converters must be deterministic and side-effect free.
- Prefer converters for reusable field-level conversion, parsing, normalization, or external-to-internal representation translation.

### Appropriate converter examples

- `String -> LocalDate`
- `Instant -> LocalDateTime`
- provider status code -> internal enum
- currency text -> normalized currency code
- provider amount representation -> `BigDecimal`
- JSON representation -> strongly typed value object
- external enum -> internal enum

### Not allowed in converters

- Repository, Redis, Feign client, service, or event calls.
- Business validation requiring current database state.
- Permission checks.
- Workflow orchestration.
- Transaction boundaries.
- Choosing a provider or business strategy.
- Silently applying accounting formulas.

### Error handling

- Reject malformed input with the existing project exception/error convention when conversion is part of an application flow.
- Return null only when null is an explicitly supported input.
- Do not silently convert unknown business values to a default enum unless the contract defines that fallback.
- Preserve raw values or fail explicitly for unknown provider codes according to the owning flow.

### Spring Converter vs domain converter

- Implement Spring `Converter<S, T>` only for framework boundary conversion, such as request parameters and configuration binding.
- Use a normal component/class for domain or provider representation conversion.
- Do not register domain converters globally when they are only valid for one provider or module.

### Naming

- Use `XxxConverter` for a reusable conversion component.
- Prefer explicit method names:
  - `toLocalDate`
  - `toInternalStatus`
  - `toBigDecimal`
  - `fromProviderCode`
- Avoid vague methods such as `convertData` or `process`.

## DTO style

- Use separate request and response DTOs.
- Do not expose JPA entities through controller responses.
- Put structural validation on request DTOs:
  - `@NotNull`
  - `@NotBlank`
  - `@Size`
  - `@PositiveOrZero`
  - custom field-level constraints
- Do not perform repository lookups or business validation inside DTOs or constraint annotations unless the validator is explicitly designed for that boundary rule.
- Do not put service, repository, client, or mapper dependencies in DTOs.
- Prefer immutable DTOs or records only when consistent with the local module.
- Use `BigDecimal` for monetary fields.
- Document externally consumed fields with `@Schema` when surrounding DTOs use Swagger.
- Use wrapper types such as `Boolean`, `Long`, and `Integer` when absence differs from zero or false.
- Request DTOs express caller intent; response DTOs express the API contract.
- Do not reuse response DTOs as persistence or provider models merely because fields currently match.

## Practical rules for future code generation

- First inspect the neighboring controller/service pair before creating a new one.
- Match the response wrapper style already used in that package.
- If the package already uses interface plus `impl`, keep it.
- If the flow is complex, prefer a facade service plus small helpers over putting branching in controller.
- Put provider and hub decision logic in service/helper classes, not in DTOs or controllers.
- When a rule affects both manual and import flows, look for a shared helper instead of copy-paste.
## Layer ownership decision

Use the following order when deciding where code belongs:

1. Does it make a business decision or require current application state?
  - Put it in service, facade, strategy, or a business helper.

2. Does it transform a complete source object into a target object?
  - Put it in a mapper.

3. Does it transform one value or one narrowly scoped representation?
  - Put it in a converter.

4. Is it generic, stateless, and unrelated to the business domain?
  - Consider a utility class.

5. Does it validate request shape?
  - Put it in DTO bean validation or a boundary validator.

6. Does it validate a business rule?
  - Put it in service or a dedicated business validator.
