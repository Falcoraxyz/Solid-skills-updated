# NestJS Applications

How to apply SOLID, TDD, and clean architecture in NestJS. Use this when the project uses NestJS; the principles are the same as elsewhere, only the wiring differs.

## Mapping Principles to NestJS

| Principle | In NestJS |
|-----------|-----------|
| SRP | One job per provider; controllers only translate HTTP, services hold use-case logic |
| OCP | Add new behavior with a new provider, not by editing a switch in an existing one |
| LSP | Any provider bound to a token must honor the contract of its interface |
| ISP | Small, focused interfaces per consumer, not one giant `IUserService` |
| DIP | Inject abstractions via tokens; bind implementations in the module |

## Vertical Slices: One Module per Feature

```
src/
└── orders/
    ├── orders.module.ts
    ├── orders.controller.ts          # HTTP only
    ├── place-order.use-case.ts       # application logic
    ├── order.entity.ts               # domain
    ├── order-repository.ts           # abstraction (port)
    ├── typeorm-order.repository.ts   # implementation (adapter)
    └── dto/
        └── place-order.dto.ts
```

- Keep everything for a feature together, not split into global `controllers/`, `services/`, `entities/` folders.
- Export from a module only what other modules truly need.
- Avoid circular module dependencies. If you reach for `forwardRef`, treat it as a design smell: extract the shared piece into its own module or invert the dependency with an event.

## Thin Controllers

Controllers validate input, call one use case, and map the result. No business rules.

```typescript
@Controller('orders')
export class OrdersController {
  constructor(private readonly placeOrder: PlaceOrderUseCase) {}

  @Post()
  async create(@Body() dto: PlaceOrderDto): Promise<OrderResponse> {
    const order = await this.placeOrder.execute(dto.toCommand());
    return OrderResponse.from(order);
  }
}
```

Smell: a controller with `if` statements about business state, or one that talks to a repository directly.

## Dependency Inversion with Injection Tokens

TypeScript interfaces disappear at runtime, so NestJS needs a token to inject an abstraction. Use an abstract class (works as both type and token) or a `Symbol`.

```typescript
// Port: abstract class doubles as the injection token
export abstract class OrderRepository {
  abstract save(order: Order): Promise<void>;
  abstract findById(id: OrderId): Promise<Order | null>;
}

// Adapter
@Injectable()
export class TypeOrmOrderRepository extends OrderRepository {
  /* ... */
}

// Module binds abstraction -> implementation
@Module({
  controllers: [OrdersController],
  providers: [
    PlaceOrderUseCase,
    { provide: OrderRepository, useClass: TypeOrmOrderRepository },
  ],
})
export class OrdersModule {}
```

The use case depends on `OrderRepository`, never on TypeORM, Prisma, or Mongoose. Swapping the database, or using an in-memory fake in tests, needs no change to the use case.

If you prefer interfaces, use a symbol token:

```typescript
export const ORDER_REPOSITORY = Symbol('ORDER_REPOSITORY');

constructor(@Inject(ORDER_REPOSITORY) private readonly orders: OrderRepository) {}
```

## Use Cases Instead of Fat Services

A `OrdersService` with 15 methods tends to become a god class (SRP violation, many reasons to change). Prefer one provider per use case, each with a single `execute` method.

```typescript
@Injectable()
export class PlaceOrderUseCase {
  constructor(
    private readonly orders: OrderRepository,
    private readonly payments: PaymentGateway,
  ) {}

  async execute(command: PlaceOrderCommand): Promise<Order> {
    const order = Order.place(command.customerId, command.items);
    await this.payments.authorize(order.total());
    await this.orders.save(order);
    return order;
  }
}
```

For small CRUD features a single service is fine. Split when it grows beyond one responsibility.

## Validation at the Boundary, Rules in the Domain

- DTOs with `class-validator` check shape and format of incoming data (is it a string, is it an email).
- Domain objects and value objects enforce business rules (an order needs at least one item; money cannot be negative).
- Do not rely on DTO validation alone for invariants. The domain must stay valid when called from a queue consumer, a CLI, or a test.

Keep DTOs separate from domain entities. Map between them at the edge.

## Errors

- Throw domain-specific errors from the domain and use cases (`OrderAlreadyPaidError`), not `HttpException`.
- Translate them to HTTP in an exception filter, so the domain does not know about HTTP.

```typescript
@Catch(OrderAlreadyPaidError)
export class OrderAlreadyPaidFilter implements ExceptionFilter {
  catch(error: OrderAlreadyPaidError, host: ArgumentsHost) {
    host.switchToHttp().getResponse().status(409).json({ message: error.message });
  }
}
```

## Configuration

- Inject configuration through `ConfigService` or a typed config provider, not `process.env` scattered across classes.
- Validate environment variables once at startup.

## Testing in NestJS (TDD)

Prefer testing most logic without the NestJS container at all. A use case is a plain class: construct it with fakes.

```typescript
describe('PlaceOrderUseCase', () => {
  it('when payment is authorized, saves the order', async () => {
    const orders = new InMemoryOrderRepository();
    const payments = new FakePaymentGateway({ authorizes: true });
    const useCase = new PlaceOrderUseCase(orders, payments);

    const order = await useCase.execute(aPlaceOrderCommand());

    expect(await orders.findById(order.id)).toEqual(order);
  });
});
```

Use `Test.createTestingModule` when you need to verify wiring, guards, pipes, or interceptors:

```typescript
const moduleRef = await Test.createTestingModule({
  controllers: [OrdersController],
  providers: [
    PlaceOrderUseCase,
    { provide: OrderRepository, useClass: InMemoryOrderRepository },
    { provide: PaymentGateway, useValue: fakePaymentGateway },
  ],
}).compile();

const controller = moduleRef.get(OrdersController);
```

Test layers:

| Layer | Tool | What to cover |
|-------|------|---------------|
| Domain / use cases | Plain Jest, in-memory fakes | Business rules, most tests live here |
| Controllers, guards, pipes | `Test.createTestingModule` | Wiring, validation, auth |
| HTTP end-to-end | `supertest` against `app.getHttpServer()` | A few critical flows |
| Repositories | Integration test against a real test DB | Queries and mappings |

Prefer hand-written in-memory fakes over heavy mocking: they survive refactors and make tests read like behavior.

## NestJS Warning Signs

- Controller imports a repository or ORM entity
- Provider injects `@InjectRepository(...)` directly in business logic that should be ORM-agnostic
- Service with many unrelated methods and 6+ constructor dependencies
- `forwardRef` or circular imports between modules
- `process.env` read outside the config layer
- Business rules inside DTOs, pipes, or guards
- Tests that need the full app to check a pure business rule
