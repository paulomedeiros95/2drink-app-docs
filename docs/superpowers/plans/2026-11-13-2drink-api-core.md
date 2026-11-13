# 2DRINK API Core — Plano de Implementação (Plano 1 de 4)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Entregar a API do 2DRINK com contas, bares (Google Places), check-in com selfie e bebidas, extrato imutável, badges, temporadas, ligas, ranking e Troféu Dourado. É a base que o app mobile consome.

**Architecture:**
- ASP.NET Core (.NET 10) com minimal APIs, organizada por feature.
- As regras puras ficam em `TwoDrink.Domain`, sem dependências e com testes unitários. Persistência, Google e Blob ficam em `TwoDrink.Infrastructure`.
- PostgreSQL via EF Core. O extrato é uma tabela só de inserção, protegida por permissão de banco e por trigger, com cadeia de hash por conta.
- Fotos vão direto do app para o Azure Blob, por link temporário emitido pela API.

**Tech Stack:**
- .NET 10, ASP.NET Core minimal APIs, EF Core 10 + Npgsql, PostgreSQL 17;
- xUnit, Testcontainers (PostgreSQL, Azurite), `Microsoft.Extensions.TimeProvider.Testing`;
- `Google.Apis.Auth` (token Google), `Microsoft.IdentityModel.Protocols.OpenIdConnect` (token Apple);
- `Azure.Storage.Blobs`, Google Places API (New).

**Spec:** `docs/superpowers/specs/2026-11-13-2drink-mvp-design.md` (no repo da API: `docs/docs/superpowers/specs/...` via submodule)

**Os 4 planos:**
1. **Este:** API core.
2. API social: feed, fotos, reações, comentários, filtro de palavrões, denúncia e bloqueio, push, exclusão de conta, capa de liga, evento Fitness.
3. App mobile (Expo).
4. Painel admin (Blazor) e textos legais.

O plano 3 começa assim que as Tasks 1–10 deste plano estiverem prontas, em paralelo ao plano 2, porque o teste fechado do Google precisa começar até meados de dezembro.

## Global Constraints

**Repositório e projeto**
- Repo `C:\Source\Projects\2DRINK\api\2drink-api`. Solution `TwoDrink.sln`. Namespaces `TwoDrink.*`.
- O nome do produto em textos é **2DRINK**.

**Regras sempre no servidor**
- O cliente só envia dados brutos: posição, precisão, flag de mock e bebidas.
- O horário vem sempre do `TimeProvider` do servidor, nunca do aparelho.
- Fuso de negócio: `America/Sao_Paulo`. As datas de liga e temporada são `DateOnly` e cobrem `[início 00:00, fim+1 00:00)` nesse fuso.

**Bares e check-in**
- Pontos do badge por tier: Default 1, Bronze 3, Prata 5, Ouro 10, Diamante 25, Epic 50.
- Pontos por bebida:
  - **alcoólicas** (`Beer`, `Drink`, `Gin`, `Shot`, `Wine`): 1;
  - **não alcoólicas** (`Water`, `Juice`, `Soda`, `Whey`): 1,5.
- Pontos são `decimal`, com uma casa decimal.
- Raio: 70m, mais a margem `min(precisão informada, 50m)`.
- Intervalos de check-in: 15h no mesmo bar; 1h em qualquer bar.
- Bebidas por check-in: no mínimo 1 e no máximo 15, somando os complementos.
- Os complementos são aceitos até 12h depois do check-in ou até o próximo check-in da pessoa, o que vier primeiro.
- Meta de hidratação: `águas ≥ max(1, ⌈alcoólicas/2⌉)`, contando só `Water`. Vale +2 pontos, lançados quando o check-in fecha.

**Ligas**
- Limite de 2 ligas ativas ou futuras por pessoa (configurável). Ligas encerradas não contam.
- Convite: uso único, expira em 7 dias, só o criador gera.
- Sair de uma liga é definitivo. O criador não pode sair de uma liga ativa.
- Desempate do troféu: estabelecimentos distintos com check-in no período. Se o empate persistir, todos os empatados ganham.

**Contas e segurança**
- Idade mínima: 18 anos, calculada na data de hoje em `America/Sao_Paulo`.
- Todos os limites ficam em `RulesOptions` (seção `Rules` do `appsettings.json`), nunca espalhados no código. O painel do plano 4 passa a editá-los.
- Erros de regra retornam `ProblemDetails` (HTTP 422 ou 409) com a extensão `code` em snake_case (ex.: `out_of_range`). O app traduz pelo `code`, porque o app é PT/EN.
- A chave do Google e as connection strings ficam fora do repo: user-secrets em dev, Key Vault em produção.

## Review Focus

1. **Precisão de GPS absurda** (ex.: `accuracyM = 800`): a margem para em 50m. Ninguém faz check-in a 870m do bar. *(Task 4)*
2. **Toque duplo em "confirmar"**, ou repetição pela rede: a mesma intenção gera um único check-in e um único conjunto de lançamentos. *(Task 9)*
3. **Duas pessoas aceitando o mesmo convite ao mesmo tempo:** só uma entra; a outra recebe `invite_used`. *(Task 11)*
4. **Limites de data em São Paulo:** um check-in às 23:30 do último dia da liga conta; um às 00:01 do dia seguinte não conta. *(Task 12)*
5. **Lançamentos simultâneos da mesma conta** (check-in e complemento quase juntos): a cadeia de hash não bifurca e a verificação continua válida. *(Task 5)*

---

## File Structure

```
2drink-api/
  TwoDrink.sln
  CLAUDE.md                                   aponta para docs/ (spec + planos)
  docs/                                       submodule → 2drink-app-docs
  src/TwoDrink.Domain/
    Rules/RulesOptions.cs                     todos os limites configuráveis
    Rules/BusinessTime.cs                     fuso America/Sao_Paulo, DateOnly → intervalo UTC
    Scoring/DrinkType.cs, DrinkCatalog.cs     tipos, categoria, pontos
    Scoring/Tier.cs                           enum + pontos do badge
    Scoring/HydrationGoal.cs
    Badges/BadgeEligibility.cs
    Presence/Geofence.cs
    Presence/CheckInCooldown.cs
    Leagues/LeagueStandings.cs                ranking + desempate + campeões
    Errors/RuleViolation.cs                   exceção com Code
  src/TwoDrink.Infrastructure/
    Data/AppDbContext.cs, Data/Entities/*.cs, Data/Migrations/*
    Ledger/LedgerWriter.cs, LedgerHasher.cs, LedgerVerifier.cs
    Places/IPlacesClient.cs, GooglePlacesClient.cs
    Storage/IPhotoStorage.cs, BlobPhotoStorage.cs
    Auth/IExternalTokenValidator.cs, GoogleTokenValidator.cs, AppleTokenValidator.cs, JwtIssuer.cs
  src/TwoDrink.Api/
    Program.cs
    Features/Health/, Auth/, Me/, Venues/, CheckIns/, Leagues/
    Jobs/CheckInCloser.cs, Jobs/TrophyAwarder.cs
  tests/TwoDrink.Domain.Tests/
  tests/TwoDrink.Api.Tests/
    Infrastructure/ApiFactory.cs              WebApplicationFactory + Postgres/Azurite Testcontainers + FakeTimeProvider + fakes
```

---

### Task 0: Contas e ambiente (manual, feito pelo dono do produto)

Este passo não tem código. É o caminho crítico do prazo, por isso vem primeiro.

- [ ] **Apple Developer:** conta pessoa física (US$ 99/ano) em developer.apple.com/programs/enroll.
- [ ] **Google Play Console:** conta pessoal (US$ 25). Começar já a recrutar **12 ou mais testadores** com conta Google, para o teste fechado de 14 dias que precisa começar até meados de dezembro.
- [ ] **Azure:** criar a assinatura.
- [ ] **Google Cloud:** criar o projeto, ativar a **Places API (New)**, gerar a chave restrita a essa API e configurar um alerta de orçamento.
- [ ] **Login com Google:** criar os OAuth Client IDs (iOS, Android e Web) no Google Cloud.
- [ ] **Login com Apple:** criar o Service ID no Apple Developer.
- [ ] **Máquina de desenvolvimento:**
  - instalar o .NET 10 SDK (`winget install Microsoft.DotNet.SDK.10`);
  - instalar o Docker Desktop (usado pelo Testcontainers);
  - verificar com `dotnet --list-sdks` (deve listar `10.*`) e `docker info` (deve rodar sem erro).
- [ ] **Git:** `git config --global user.name` e `user.email`.

---

### Task 1: Esqueleto da solution, submodule de docs e health check

**Files:**
- Create: `TwoDrink.sln`, os 3 projetos de `src/` e os 2 de `tests/`, `CLAUDE.md`, `.gitmodules` (submodule `docs` → repo `2drink-app-docs`)
- Create: `src/TwoDrink.Api/Program.cs`, `src/TwoDrink.Api/Features/Health/HealthEndpoints.cs`
- Create: `tests/TwoDrink.Api.Tests/Infrastructure/ApiFactory.cs`
- Test: `tests/TwoDrink.Api.Tests/HealthTests.cs`

**Interfaces:**
- Produces:
  - `ApiFactory : WebApplicationFactory<Program>, IAsyncLifetime`, que sobe um PostgreSQL via Testcontainers e expõe `FakeTimeProvider Clock` (inicial: `2027-02-06T20:00:00-03:00`), `HttpClient CreateClient()` e `AppDbContext NewDb()`.
  - `public partial class Program`.
  - O rate limiter global particionado por usuário (ou IP, se anônimo): 120 requisições por minuto.

- [ ] **Step 1:** Criar a solution e os projetos (`dotnet new sln`; `classlib` para Domain e Infrastructure; `web` para Api; `xunit` para os testes). Configurar as referências: Api → Infrastructure → Domain.
- [ ] **Step 2:** Adicionar o submodule: `git submodule add <url do 2drink-app-docs> docs`. Escrever o `CLAUDE.md`: o produto é o 2DRINK; a spec está em `docs/docs/superpowers/specs/2026-11-13-2drink-mvp-design.md` e os planos em `docs/docs/superpowers/plans/`; as regras de negócio vivem em `TwoDrink.Domain`; os erros usam o `code` de `ProblemDetails`.
- [ ] **Step 3:** Escrever o teste que falha:

```csharp
[Fact] public async Task Health_returns_ok() {
    var res = await _factory.CreateClient().GetAsync("/health");
    Assert.Equal(HttpStatusCode.OK, res.StatusCode);
}
```

- [ ] **Step 4:** Rodar `dotnet test --filter Health_returns_ok`. Esperado: FAIL (404 ou falha de compilação).
- [ ] **Step 5:** Implementar `MapHealthEndpoints()`, o rate limiter e o `ProblemDetails`. Montar o `ApiFactory`: o container `postgres:17`, a migration aplicada no `InitializeAsync` e o `TimeProvider` substituído pelo `FakeTimeProvider`.
- [ ] **Step 6:** Rodar `dotnet test`. Esperado: PASS.
- [ ] **Step 7:** Commit: `chore: scaffold solution, docs submodule and health check`.

---

### Task 2: Regras de pontuação: bebidas, tiers e hidratação

**Files:**
- Create: `src/TwoDrink.Domain/Rules/RulesOptions.cs`, `Scoring/DrinkType.cs`, `Scoring/DrinkCatalog.cs`, `Scoring/Tier.cs`, `Scoring/HydrationGoal.cs`, `Errors/RuleViolation.cs`
- Test: `tests/TwoDrink.Domain.Tests/ScoringTests.cs`

**Interfaces:**
- Produces:
  - `enum DrinkType { Beer, Drink, Gin, Shot, Wine, Water, Juice, Soda, Whey }`
  - `enum DrinkCategory { Alcoholic, NonAlcoholic }`
  - `record DrinkLine(DrinkType Type, int Quantity)`
  - `DrinkCatalog(RulesOptions o)` com:
    - `DrinkCategory CategoryOf(DrinkType t)`
    - `decimal PointsFor(DrinkLine l)`
  - `enum Tier { Default = 0, Bronze = 1, Prata = 2, Ouro = 3, Diamante = 4, Epic = 5 }` (a ordem importa: maior = melhor)
  - `static decimal TierPoints.Of(Tier t)`
  - `static bool HydrationGoal.IsMet(IEnumerable<DrinkLine> lines)`
  - `class RuleViolation(string code, string message) : Exception`, com a propriedade `Code`
  - `record RulesOptions`, com os defaults da spec:

```csharp
public sealed record RulesOptions {
    public double RadiusMeters { get; init; } = 70;
    public double MaxAccuracyMarginMeters { get; init; } = 50;
    public TimeSpan SameVenueCooldown { get; init; } = TimeSpan.FromHours(15);
    public TimeSpan AnyVenueCooldown { get; init; } = TimeSpan.FromHours(1);
    public TimeSpan ComplementWindow { get; init; } = TimeSpan.FromHours(12);
    public int MaxDrinksPerCheckIn { get; init; } = 15;
    public decimal AlcoholicPoints { get; init; } = 1m;
    public decimal NonAlcoholicPoints { get; init; } = 1.5m;
    public decimal HydrationBonus { get; init; } = 2m;
    public int MaxActiveLeaguesPerUser { get; init; } = 2;
    public TimeSpan InviteLifetime { get; init; } = TimeSpan.FromDays(7);
}
```

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Theory]
[InlineData(DrinkType.Beer, 3, 3.0)] [InlineData(DrinkType.Gin, 1, 1.0)]
[InlineData(DrinkType.Water, 5, 7.5)] [InlineData(DrinkType.Soda, 2, 3.0)] [InlineData(DrinkType.Whey, 1, 1.5)]
public void Points_per_drink(DrinkType t, int q, decimal expected) =>
    Assert.Equal(expected, new DrinkCatalog(new()).PointsFor(new(t, q)));

[Theory]
[InlineData(Tier.Default, 1)] [InlineData(Tier.Bronze, 3)] [InlineData(Tier.Prata, 5)]
[InlineData(Tier.Ouro, 10)] [InlineData(Tier.Diamante, 25)] [InlineData(Tier.Epic, 50)]
public void Tier_points(Tier t, decimal p) => Assert.Equal(p, TierPoints.Of(t));

[Fact] public void Hydration_needs_at_least_one_water() =>
    Assert.False(HydrationGoal.IsMet([new(DrinkType.Juice, 3)]));
[Fact] public void Hydration_one_water_alone_is_met() =>
    Assert.True(HydrationGoal.IsMet([new(DrinkType.Water, 1)]));
[Fact] public void Hydration_three_beers_needs_two_waters() {
    Assert.False(HydrationGoal.IsMet([new(DrinkType.Beer, 3), new(DrinkType.Water, 1)]));
    Assert.True(HydrationGoal.IsMet([new(DrinkType.Beer, 3), new(DrinkType.Water, 2)]));
}
[Fact] public void Hydration_juice_does_not_count_as_water() =>
    Assert.False(HydrationGoal.IsMet([new(DrinkType.Beer, 2), new(DrinkType.Juice, 1)]));
```

- [ ] **Step 2:** Rodar `dotnet test tests/TwoDrink.Domain.Tests`. Esperado: FAIL (tipos não existem).
- [ ] **Step 3:** Implementar. Água, suco, refrigerante e whey são `NonAlcoholic`; o resto é `Alcoholic`.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(domain): drink, tier and hydration scoring rules`.

---

### Task 3: Elegibilidade de badge

**Files:**
- Create: `src/TwoDrink.Domain/Badges/BadgeEligibility.cs`
- Test: `tests/TwoDrink.Domain.Tests/BadgeEligibilityTests.cs`

**Interfaces:**
- Consumes: `Tier` (Task 2).
- Produces: `static bool BadgeEligibility.ShouldGrant(Tier currentTier, Tier? highestHeldThisSeason)`.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Fact] public void First_visit_in_season_grants() => Assert.True(BadgeEligibility.ShouldGrant(Tier.Default, null));
[Fact] public void Same_tier_does_not_grant() => Assert.False(BadgeEligibility.ShouldGrant(Tier.Bronze, Tier.Bronze));
[Fact] public void Upgrade_grants_evolution() => Assert.True(BadgeEligibility.ShouldGrant(Tier.Ouro, Tier.Bronze));
[Fact] public void Downgrade_does_not_grant() => Assert.False(BadgeEligibility.ShouldGrant(Tier.Bronze, Tier.Epic));
[Fact] public void Renewal_in_same_season_with_lower_or_equal_tier_does_not_grant() {
    Assert.False(BadgeEligibility.ShouldGrant(Tier.Ouro, Tier.Epic));
    Assert.False(BadgeEligibility.ShouldGrant(Tier.Epic, Tier.Epic));
}
```

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar: `highest is null || current > highest`.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(domain): badge eligibility with evolution and no-downgrade`.

---

### Task 4: Presença (geofence) e intervalos de check-in

**Files:**
- Create: `src/TwoDrink.Domain/Presence/Geofence.cs`, `Presence/CheckInCooldown.cs`, `Rules/BusinessTime.cs`
- Test: `tests/TwoDrink.Domain.Tests/PresenceTests.cs`, `BusinessTimeTests.cs`

**Interfaces:**
- Consumes: `RulesOptions`, `RuleViolation` (Task 2).
- Produces:
  - `record GeoPoint(double Lat, double Lng)`
  - `record DeviceFix(GeoPoint Point, double AccuracyMeters, bool IsMocked)`
  - `Geofence(RulesOptions o)`:
    - `void EnsureInside(DeviceFix fix, GeoPoint venue)`: lança `RuleViolation` com `mock_location` ou `out_of_range`;
    - `static double DistanceMeters(GeoPoint a, GeoPoint b)` (haversine, raio da Terra 6 371 000 m).
  - `CheckInCooldown(RulesOptions o)`:
    - `void Ensure(DateTimeOffset now, DateTimeOffset? lastAtThisVenue, DateTimeOffset? lastAnywhere)`: lança `cooldown_same_venue` ou `cooldown_any_venue`.
  - `static class BusinessTime`:
    - `TimeZoneInfo Zone` (`America/Sao_Paulo`);
    - `(DateTimeOffset StartUtc, DateTimeOffset EndUtcExclusive) Range(DateOnly start, DateOnly end)`;
    - `DateOnly Today(DateTimeOffset now)`.

- [ ] **Step 1:** Escrever os testes que falham. Os pontos usam deslocamentos para o norte (1 grau de latitude ≈ 111 195 m):

```csharp
static readonly GeoPoint Bar = new(-23.5505, -46.6333);
static GeoPoint North(double m) => new(Bar.Lat + m / 111_195.0, Bar.Lng);
Geofence G => new(new RulesOptions());

[Fact] public void Inside_70m_ok() => G.EnsureInside(new(North(69), 5, false), Bar);
[Fact] public void At_100m_with_40m_accuracy_ok() => G.EnsureInside(new(North(100), 40, false), Bar);
[Fact] public void Accuracy_margin_capped_at_50m() {
    var ex = Assert.Throws<RuleViolation>(() => G.EnsureInside(new(North(130), 800, false), Bar));
    Assert.Equal("out_of_range", ex.Code);
}
[Fact] public void Mocked_location_rejected_even_inside() =>
    Assert.Equal("mock_location", Assert.Throws<RuleViolation>(() => G.EnsureInside(new(North(1), 5, true), Bar)).Code);

[Fact] public void Same_venue_within_15h_rejected() {
    var now = DateTimeOffset.Parse("2027-02-07T10:00:00-03:00");
    var ex = Assert.Throws<RuleViolation>(() => new CheckInCooldown(new()).Ensure(now, now.AddHours(-14.9), now.AddHours(-14.9)));
    Assert.Equal("cooldown_same_venue", ex.Code);
}
[Fact] public void Other_venue_within_1h_rejected() {
    var now = DateTimeOffset.Parse("2027-02-07T10:00:00-03:00");
    Assert.Equal("cooldown_any_venue",
        Assert.Throws<RuleViolation>(() => new CheckInCooldown(new()).Ensure(now, null, now.AddMinutes(-59))).Code);
}
[Fact] public void Other_venue_after_1h_ok() {
    var now = DateTimeOffset.Parse("2027-02-07T10:00:00-03:00");
    new CheckInCooldown(new()).Ensure(now, now.AddHours(-16), now.AddMinutes(-61));
}

[Fact] public void Range_covers_end_day_until_midnight_sao_paulo() {
    var (s, e) = BusinessTime.Range(new(2027, 2, 1), new(2027, 2, 15));
    Assert.Equal(DateTimeOffset.Parse("2027-02-01T03:00:00Z"), s);
    Assert.Equal(DateTimeOffset.Parse("2027-02-16T03:00:00Z"), e);
}
```

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar. Aceitar quando `distância ≤ RadiusMeters + min(Accuracy, MaxAccuracyMarginMeters)`. A checagem de mock vem antes da de distância.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(domain): geofence, cooldowns and business time zone`.

---

### Task 5: Extrato imutável (ledger)

**Files:**
- Create: `src/TwoDrink.Infrastructure/Data/AppDbContext.cs`, `Data/Entities/LedgerAccount.cs`, `Data/Entities/LedgerEntry.cs`
- Create: `src/TwoDrink.Infrastructure/Ledger/LedgerWriter.cs`, `LedgerHasher.cs`, `LedgerVerifier.cs`
- Create: a migration `InitialLedger`, com SQL bruto para a trigger e para as permissões
- Test: `tests/TwoDrink.Api.Tests/LedgerTests.cs`

**Interfaces:**
- Produces:
  - `LedgerAccount { Guid Id }`. É o identificador pseudônimo da conta. O usuário aponta para ele; o extrato nunca aponta para o usuário. É isso que permite anonimizar na exclusão de conta.
  - `enum LedgerEntryType { CheckIn, Drink, Badge, HydrationBonus, Reversal }`
  - `LedgerEntry`, com os campos:
    - `long Id`, `Guid AccountId`, `long Sequence` (1, 2, 3… por conta), `LedgerEntryType Type`;
    - `Guid? VenueId`, `Guid? CheckInId`, `DrinkType? DrinkType`, `int? Quantity`, `Tier? Tier`;
    - `decimal Points`, `long? ReversesEntryId`, `string? Reason`, `string? ActorAdmin`;
    - `DateTimeOffset CreatedAt`, `string PrevHash`, `string Hash`.
  - `record NewEntry(...)`: os mesmos campos, menos `Id`, `Sequence`, `CreatedAt`, `PrevHash` e `Hash`.
  - `LedgerWriter(AppDbContext db, TimeProvider clock)`:
    - `Task<IReadOnlyList<LedgerEntry>> AppendAsync(Guid accountId, IReadOnlyList<NewEntry> entries, CancellationToken ct)`.
    - Deve ser chamado dentro da transação de quem chama. Começa com `SELECT pg_advisory_xact_lock(hashtext(accountId::text))`.
  - `static string LedgerHasher.Compute(LedgerEntry e)`: SHA-256 em hex de `AccountId|Sequence|Type|VenueId|CheckInId|DrinkType|Quantity|Tier|Points:F1|ReversesEntryId|CreatedAt(UTC ticks)|PrevHash`. Não inclui nenhum dado pessoal. O primeiro `PrevHash` de cada conta é `"GENESIS"`.
  - `LedgerVerifier(AppDbContext db)`: `Task<IReadOnlyList<Guid>> FindBrokenChainsAsync(CancellationToken ct)`.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Fact] public async Task Append_chains_hashes_per_account() {
    var acc = await Seed.Account(db);
    var e = await Append(acc, Drink(DrinkType.Beer, 2), Drink(DrinkType.Water, 1));
    Assert.Equal([1L, 2L], e.Select(x => x.Sequence));
    Assert.Equal("GENESIS", e[0].PrevHash);
    Assert.Equal(e[0].Hash, e[1].PrevHash);
    Assert.Empty(await new LedgerVerifier(db).FindBrokenChainsAsync(default));
}
[Fact] public async Task Update_is_rejected_by_database_trigger() {
    var e = (await Append(await Seed.Account(db), Drink(DrinkType.Beer, 1)))[0];
    var ex = await Assert.ThrowsAsync<PostgresException>(() =>
        db.Database.ExecuteSqlRawAsync($"UPDATE ledger_entries SET points = 50 WHERE id = {e.Id}"));
    Assert.Equal("P0001", ex.SqlState);
}
[Fact] public async Task Delete_is_rejected_for_app_role() { /* conectar como twodrink_app; DELETE → SqlState 42501 */ }
[Fact] public async Task Verifier_detects_tampering() {
    // Desliga a trigger como superusuário, altera points e religa.
    // FindBrokenChainsAsync deve retornar a conta adulterada.
}
[Fact] public async Task Concurrent_appends_do_not_fork_chain() {
    var acc = await Seed.Account(db);
    await Task.WhenAll(Enumerable.Range(0, 10).Select(_ => AppendInNewScope(acc, Drink(DrinkType.Beer, 1))));
    var seqs = await db.LedgerEntries.Where(x => x.AccountId == acc).Select(x => x.Sequence).OrderBy(x => x).ToListAsync();
    Assert.Equal(Enumerable.Range(1, 10).Select(i => (long)i), seqs);
    Assert.Empty(await new LedgerVerifier(db).FindBrokenChainsAsync(default));
}
```

- [ ] **Step 2:** Rodar `dotnet test --filter LedgerTests`. Esperado: FAIL.
- [ ] **Step 3:** Implementar. A migration tem:
  - um índice único `(account_id, sequence)`;
  - uma trigger `BEFORE UPDATE OR DELETE` que faz `RAISE EXCEPTION 'ledger is append-only'`;
  - a criação do papel `twodrink_app` com `GRANT SELECT, INSERT` em `ledger_entries` e `REVOKE UPDATE, DELETE`.

  Em produção, a API conecta como `twodrink_app`, e as migrations rodam com o dono do banco. O `ApiFactory` usa a mesma separação.

  Registrar também um `BackgroundService` diário que chama `FindBrokenChainsAsync` e, se houver contas na lista, registra um log `Critical` com elas. O alerta no Azure é ligado a esse log.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(ledger): append-only hash-chained ledger with db enforcement`.

---

### Task 6: Login (Google e Apple), tokens próprios e perfil 18+

**Files:**
- Create: `src/TwoDrink.Infrastructure/Auth/IExternalTokenValidator.cs`, `GoogleTokenValidator.cs`, `AppleTokenValidator.cs`, `JwtIssuer.cs`
- Create: `Data/Entities/User.cs`, `Data/Entities/RefreshToken.cs`, uma migration
- Create: `src/TwoDrink.Api/Features/Auth/AuthEndpoints.cs`, `Features/Me/ProfileEndpoints.cs`
- Test: `tests/TwoDrink.Api.Tests/AuthTests.cs`

**Interfaces:**
- Consumes: `LedgerAccount` (Task 5), `BusinessTime.Today` (Task 4).
- Produces:
  - `enum ExternalProvider { Google, Apple }`
  - `record ExternalIdentity(ExternalProvider Provider, string Subject, string? Email)`
  - `interface IExternalTokenValidator { Task<ExternalIdentity> ValidateAsync(ExternalProvider p, string token, CancellationToken ct); }`. Nos testes é substituído por um fake que aceita `"valid:<subject>"`.
  - `User`:
    - `Guid Id`, `ExternalProvider Provider`, `string Subject` (único por provider), `Guid LedgerAccountId`;
    - `string? DisplayName`, `DateOnly? BirthDate`, `string? AcceptedTermsVersion`, `string Locale` (`pt-BR` ou `en`);
    - `bool ProfileComplete`, verdadeiro quando os três campos do perfil estão preenchidos.
  - Endpoints:
    - `POST /auth/google` `{ idToken }` e `POST /auth/apple` `{ identityToken }`, que devolvem `{ accessToken, refreshToken, profileComplete }`;
    - `POST /auth/refresh` `{ refreshToken }`, com rotação do refresh token;
    - `PUT /me/profile` `{ displayName, birthDate, acceptedTermsVersion, locale }`.
  - Tokens:
    - access token JWT de 1h, com claims `sub` = `User.Id` e `ledger` = `LedgerAccountId`;
    - refresh token de 60 dias, guardado como hash SHA-256.
  - Policy `"Player"`: exige `ProfileComplete`. Todos os endpoints das Tasks 7 a 13 usam essa policy.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Fact] public async Task First_login_creates_user_with_incomplete_profile() {
    var r = await PostJson<AuthResponse>("/auth/google", new { idToken = "valid:g-123" });
    Assert.False(r.ProfileComplete);
}
[Fact] public async Task Same_subject_logs_into_same_user() { /* login duas vezes → mesmo sub no JWT */ }
[Fact] public async Task Underage_profile_is_rejected() {
    Clock.SetUtcNow(DateTimeOffset.Parse("2027-02-06T23:00:00Z"));
    var res = await PutProfile(birthDate: new DateOnly(2009, 2, 7)); // faz 18 anos só amanhã
    await AssertProblem(res, 422, "underage");
}
[Fact] public async Task Eighteenth_birthday_today_is_accepted() { /* birthDate 2009-02-06 → 200 */ }
[Fact] public async Task Player_endpoints_require_complete_profile() { /* GET /me/leagues sem perfil completo → 403 */ }
[Fact] public async Task Invalid_token_is_401() { /* idToken "bad" → 401 */ }
[Fact] public async Task Refresh_token_is_single_use() { /* refresh duas vezes com o mesmo token → a segunda dá 401 */ }
```

- [ ] **Step 2:** Rodar `dotnet test --filter AuthTests`. Esperado: FAIL.
- [ ] **Step 3:** Implementar:
  - Google: `GoogleJsonWebSignature.ValidateAsync`, com `Audience` = os três Client IDs (vêm da configuração).
  - Apple: validar o JWT com as chaves de `https://appleid.apple.com/auth/keys`, `iss` = `https://appleid.apple.com` e `aud` = o bundle ID ou o Service ID.
  - No primeiro login, criar o `User` e o `LedgerAccount`.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(auth): google/apple sign-in, jwt, 18+ profile`.

---

### Task 7: Bares via Google Places, tiers de patrocínio e temporadas

**Files:**
- Create: `src/TwoDrink.Infrastructure/Places/IPlacesClient.cs`, `GooglePlacesClient.cs`
- Create: `Data/Entities/Venue.cs`, `VenueSponsorship.cs`, `Season.cs`, uma migration (com seed das temporadas)
- Create: `src/TwoDrink.Api/Features/Venues/VenueEndpoints.cs`, `Features/Venues/VenueTierService.cs`, `Features/Seasons/SeasonService.cs`
- Test: `tests/TwoDrink.Api.Tests/VenueTests.cs`, `tests/TwoDrink.Api.Tests/GooglePlacesClientTests.cs`

**Interfaces:**
- Produces:
  - `record PlaceResult(string PlaceId, string Name, string? Address, GeoPoint Location)`
  - `interface IPlacesClient`:
    - `Task<IReadOnlyList<PlaceResult>> NearbyAsync(GeoPoint p, CancellationToken ct)`;
    - `Task<PlaceResult?> DetailsAsync(string placeId, CancellationToken ct)`.
  - `Venue`: `Guid Id`, `string PlaceId` (único), `string Name`, `string? Address`, `double Lat`, `double Lng`, `DateTimeOffset CachedAt`.
  - `VenueSponsorship`: `Guid Id`, `Guid VenueId`, `Tier Tier`, `string? BadgeArtUrl`, `DateTimeOffset StartsAt`, `DateTimeOffset EndsAt`, `string CreatedBy`.
  - `VenueTierService`: `Task<(Tier Tier, string? ArtUrl)> CurrentAsync(Guid venueId, DateTimeOffset at, CancellationToken ct)`. Sem patrocínio vigente, devolve `(Tier.Default, null)`; `null` significa a arte padrão.
  - `Venue` tem ainda `Task<Venue> EnsureFreshAsync(Venue v, CancellationToken ct)`: se `CachedAt` tem mais de 30 dias, chama `DetailsAsync` e atualiza.
  - `Season`: `Guid Id`, `string Name`, `DateOnly StartDate`, `DateOnly EndDate`.
  - `SeasonService`:
    - `Task<Season?> AtAsync(DateTimeOffset at, CancellationToken ct)`;
    - `Task<Season> CreateAsync(string name, DateOnly start, DateOnly end, CancellationToken ct)`: lança `season_overlap` se houver sobreposição.
  - Endpoint `GET /venues/nearby?lat=&lng=`, que devolve `[{ venueId, name, address, lat, lng, tier }]`.
  - Seed das temporadas (para o teste fechado de dezembro e o Carnaval):
    - "2026 – 2º semestre": 2026-07-01 a 2026-12-31;
    - "2027 – 1º semestre": 2027-01-01 a 2027-06-30.

- [ ] **Step 1:** Escrever os testes que falham (`IPlacesClient` substituído por um fake que conta as chamadas):

```csharp
[Fact] public async Task Nearby_upserts_venues_by_place_id() { /* 2 chamadas com o mesmo place → 1 Venue */ }
[Fact] public async Task Nearby_reuses_cached_results_for_same_area() {
    await GetNearby(-23.55050, -46.63330); await GetNearby(-23.55052, -46.63331);
    Assert.Equal(1, FakePlaces.NearbyCalls);
}
[Fact] public async Task Tier_is_default_without_sponsorship() { /* CurrentAsync → Default */ }
[Fact] public async Task Tier_follows_sponsorship_window() {
    // Patrocínio Epic de 2027-02-01 a 2027-02-10.
    // CurrentAsync(2027-02-05) → Epic; CurrentAsync(2027-02-11) → Default.
}
[Fact] public async Task Overlapping_season_rejected() { /* CreateAsync 2027-06-01..2027-12-31 → season_overlap */ }
[Fact] public async Task Stale_venue_is_refreshed_after_30_days() { /* CachedAt de 31 dias atrás → DetailsCalls == 1 */ }
```

`GooglePlacesClientTests` usa um `HttpMessageHandler` fake e verifica que a requisição de `NearbyAsync` é:
- `POST https://places.googleapis.com/v1/places:searchNearby`;
- headers `X-Goog-Api-Key` e `X-Goog-FieldMask: places.id,places.displayName,places.formattedAddress,places.location`;
- corpo com `includedTypes ["bar","restaurant"]`, `maxResultCount 20` e círculo de raio 300m.

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar. A chave de cache é a coordenada arredondada para 3 casas decimais (cerca de 110m). O cache é `IMemoryCache` com validade de 1h.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(venues): google places nearby with cache, sponsorship tiers, seasons`.

---

### Task 8: Intenção de check-in e upload da selfie

**Files:**
- Create: `src/TwoDrink.Infrastructure/Storage/IPhotoStorage.cs`, `BlobPhotoStorage.cs`
- Create: `Data/Entities/CheckInIntent.cs`, uma migration
- Create: `src/TwoDrink.Api/Features/CheckIns/CheckInIntentEndpoints.cs`
- Test: `tests/TwoDrink.Api.Tests/CheckInIntentTests.cs`

**Interfaces:**
- Consumes: `Geofence`, `CheckInCooldown` (Task 4); `Venue`, `EnsureFreshAsync` (Task 7).
- Produces:
  - `interface IPhotoStorage`:
    - `Uri CreateUploadUrl(string blobName, TimeSpan ttl)`: SAS só de escrita, container privado `selfies`;
    - `Task<PhotoInfo?> GetInfoAsync(string blobName, CancellationToken ct)`.
  - `record PhotoInfo(long SizeBytes, string ContentType)`
  - `CheckInIntent`: `Guid Id`, `Guid UserId`, `Guid VenueId`, `string BlobName` (`checkins/{Id}.jpg`), `DateTimeOffset CreatedAt`, `DateTimeOffset ExpiresAt` (10 min), `Guid? ConfirmedCheckInId`.
  - `POST /checkins/intents` com `{ venueId, lat, lng, accuracyM, isMocked }`:
    - valida a presença e os intervalos antes de gastar um upload;
    - devolve `{ intentId, uploadUrl, expiresAt }`.
  - O app precisa reencodar a selfie em JPEG sem EXIF antes de enviar (plano 3). A API checa o tipo e o tamanho na confirmação.

- [ ] **Step 1:** Escrever os testes que falham (Azurite via Testcontainers):

```csharp
[Fact] public async Task Intent_outside_radius_is_422_out_of_range() { }
[Fact] public async Task Intent_within_cooldown_is_422() { /* check-in confirmado há 30 min em outro bar → cooldown_any_venue */ }
[Fact] public async Task Intent_returns_writable_url_for_its_blob_only() {
    var r = await CreateIntent(InsideBar);
    Assert.Contains($"checkins/{r.IntentId}.jpg", r.UploadUrl.ToString());
    await UploadJpeg(r.UploadUrl); // PUT com x-ms-blob-type: BlockBlob → 201
}
```

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(checkins): check-in intent with presence validation and SAS upload`.

---

### Task 9: Confirmar o check-in: bebidas, badge e lançamentos

**Files:**
- Create: `Data/Entities/CheckIn.cs`, `Data/Entities/UserBadge.cs`, uma migration
- Create: `src/TwoDrink.Api/Features/CheckIns/CheckInService.cs`, `CheckInEndpoints.cs`
- Test: `tests/TwoDrink.Api.Tests/CheckInConfirmTests.cs`

**Interfaces:**
- Consumes:
  - `DrinkCatalog`, `TierPoints`, `BadgeEligibility`, `Geofence`, `CheckInCooldown` (Tasks 2–4);
  - `LedgerWriter` (Task 5);
  - `VenueTierService`, `SeasonService` (Task 7);
  - `CheckInIntent`, `IPhotoStorage` (Task 8).
- Produces:
  - `CheckIn`:
    - `Guid Id`, `Guid UserId`, `Guid LedgerAccountId`, `Guid VenueId`, `Guid? SeasonId`, `string PhotoBlobName`;
    - `DateTimeOffset CreatedAt`, `DateTimeOffset ComplementsUntil` (= `CreatedAt + 12h`), `DateTimeOffset? ClosedAt`;
    - `int DrinkCount`, `bool HydrationMet`.
  - `UserBadge`: `Guid Id`, `Guid UserId`, `Guid VenueId`, `Guid SeasonId`, `Tier Tier`, `string? ArtUrl`, `Guid CheckInId`, `DateTimeOffset GrantedAt`.
  - `POST /checkins/intents/{intentId}/confirm` com `{ lat, lng, accuracyM, isMocked, drinks: [{ type, quantity }] }`. Devolve `{ checkInId, pointsAwarded, badge: { tier, artUrl, points } | null }`.
  - `CheckInService.ConfirmAsync(...)`, numa única transação:
    1. a intenção existe, é do usuário, não expirou;
    2. se já foi confirmada, **devolve o mesmo resultado** (idempotente);
    3. presença;
    4. intervalos;
    5. bebidas: entre 1 e 15 no total, cada quantidade ≥ 1 (senão `drink_limit` ou `drinks_required`);
    6. a foto existe, é `image/jpeg` e tem até 5 MB (senão `photo_missing`);
    7. grava, numa chamada só de `AppendAsync`: `CheckIn` (0 ponto), um `Drink` por linha, e `Badge` se `ShouldGrant`, com os pontos do tier **no momento**;
    8. grava o `UserBadge`.

    O fechamento do check-in anterior entra na Task 10.
  - Se não existe temporada vigente: não dá badge, mas as bebidas pontuam.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Fact] public async Task First_visit_scores_drinks_plus_default_badge() {
    var r = await Confirm(InsideBar, (DrinkType.Water, 5));
    Assert.Equal(7.5m + 1m, r.PointsAwarded);
    Assert.Equal(Tier.Default, r.Badge!.Tier);
}
[Fact] public async Task Second_visit_same_tier_scores_only_drinks() {
    await Confirm(InsideBar, (DrinkType.Beer, 1)); Clock.Advance(TimeSpan.FromHours(16));
    var r = await Confirm(InsideBar, (DrinkType.Beer, 2));
    Assert.Equal(2m, r.PointsAwarded); Assert.Null(r.Badge);
}
[Fact] public async Task Tier_upgrade_grants_new_badge_with_new_points() {
    await Confirm(InsideBar, (DrinkType.Beer, 1));
    await Sponsor(Bar, Tier.Epic); Clock.Advance(TimeSpan.FromHours(16));
    var r = await Confirm(InsideBar, (DrinkType.Beer, 1));
    Assert.Equal(51m, r.PointsAwarded);
}
[Fact] public async Task Renewal_in_same_season_gives_no_new_badge_but_next_season_does() {
    await Sponsor(Bar, Tier.Epic); await Confirm(InsideBar, (DrinkType.Beer, 1));          // +50 +1
    await EndSponsorship(Bar); await Sponsor(Bar, Tier.Ouro); Clock.Advance(TimeSpan.FromHours(16));
    Assert.Null((await Confirm(InsideBar, (DrinkType.Beer, 1))).Badge);                  // mesma temporada
    Clock.SetUtcNow(DateTimeOffset.Parse("2027-07-02T20:00:00Z")); await SeedSeason("2027 – 2º semestre", new(2027,7,1), new(2027,12,31));
    Assert.Equal(Tier.Ouro, (await Confirm(InsideBar, (DrinkType.Beer, 1))).Badge!.Tier); // temporada nova reinicia
}
[Fact] public async Task Badge_points_are_frozen_when_tier_later_drops() {
    await Sponsor(Bar, Tier.Epic); await Confirm(InsideBar, (DrinkType.Beer, 1));
    await EndSponsorship(Bar);
    Assert.Equal(51m, await LedgerTotal(Me));
}
[Fact] public async Task Zero_drinks_rejected() { /* drinks_required */ }
[Fact] public async Task Sixteen_drinks_rejected() { /* drink_limit */ }
[Fact] public async Task Confirm_without_uploaded_photo_rejected() { /* photo_missing */ }
[Fact] public async Task Expired_intent_rejected() { /* Clock.Advance(11 min) → intent_expired */ }
[Fact] public async Task Double_confirm_is_idempotent() {
    var a = Confirm(InsideBar, (DrinkType.Beer, 1)); var b = ConfirmSameIntent();
    await Task.WhenAll(a, b);
    Assert.Equal(1, await CheckInCount(Me)); Assert.Equal(2m, await LedgerTotal(Me)); // 1 bebida + badge 1
}
[Fact] public async Task Moving_out_of_range_between_intent_and_confirm_rejected() { /* out_of_range */ }
```

- [ ] **Step 2:** Rodar `dotnet test --filter CheckInConfirmTests`. Esperado: FAIL.
- [ ] **Step 3:** Implementar. A idempotência usa `SELECT ... FOR UPDATE` na `CheckInIntent` e checa `ConfirmedCheckInId`.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(checkins): confirm check-in with drinks, badges and ledger entries`.

---

### Task 10: Complementos, fechamento do check-in e bônus de hidratação

**Files:**
- Create: `src/TwoDrink.Api/Jobs/CheckInCloser.cs` (serviço e `BackgroundService` a cada 5 min), `Features/CheckIns/ComplementEndpoints.cs`, `Features/Me/LedgerEndpoints.cs`
- Modify: `src/TwoDrink.Api/Features/CheckIns/CheckInService.cs`. Antes de gravar os lançamentos, `ConfirmAsync` chama `CheckInCloser.CloseAsync` no check-in aberto anterior do usuário, dentro da mesma transação.
- Test: `tests/TwoDrink.Api.Tests/CheckInComplementTests.cs`

**Interfaces:**
- Consumes: `CheckIn`, `CheckInService` (Task 9); `HydrationGoal` (Task 2); `LedgerWriter` (Task 5).
- Produces:
  - `POST /checkins/{id}/drinks` com `{ lat, lng, accuracyM, isMocked, drinks }`. Devolve `{ pointsAwarded, drinkCount }`.
  - `CheckInCloser.CloseAsync(Guid checkInId, CancellationToken ct)`:
    - define `ClosedAt`;
    - calcula `HydrationGoal` sobre todos os `Drink` daquele check-in;
    - se a meta foi batida, grava `HydrationBonus` (+2) e marca `HydrationMet`;
    - é idempotente.
  - `CheckInCloser.SweepAsync(CancellationToken ct)`: fecha os check-ins com `ComplementsUntil ≤ now` e `ClosedAt == null`.
  - `GET /me/ledger?before=&limit=`: o extrato da própria conta, só leitura, do mais novo para o mais antigo.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Fact] public async Task Complement_adds_drinks_without_new_photo() {
    var c = await Confirm(InsideBar, (DrinkType.Beer, 2)); Clock.Advance(TimeSpan.FromHours(2));
    var r = await Complement(c, InsideBar, (DrinkType.Water, 1));
    Assert.Equal(1.5m, r.PointsAwarded); Assert.Equal(3, r.DrinkCount);
}
[Fact] public async Task Complement_out_of_range_rejected_and_checkin_stays_open() {
    var c = await Confirm(InsideBar, (DrinkType.Beer, 1));
    await AssertProblem(await ComplementRaw(c, North(300), (DrinkType.Beer, 1)), 422, "out_of_range");
    Assert.Equal(200, (int)(await ComplementRaw(c, InsideBar, (DrinkType.Beer, 1))).StatusCode);
}
[Fact] public async Task Complement_cannot_exceed_15_total() { /* 14 + 2 → drink_limit */ }
[Fact] public async Task Complement_after_12h_rejected() { /* checkin_closed */ }
[Fact] public async Task New_checkin_elsewhere_closes_previous() { /* complemento no anterior → checkin_closed */ }
[Fact] public async Task Hydration_bonus_on_close_when_goal_met() {
    var c = await Confirm(InsideBar, (DrinkType.Beer, 2)); await Complement(c, InsideBar, (DrinkType.Water, 1));
    Clock.Advance(TimeSpan.FromHours(12)); await Sweep();
    Assert.Equal(2m + 1.5m + 1m + 2m, await LedgerTotal(Me));
}
[Fact] public async Task No_hydration_bonus_when_goal_missed() { }
[Fact] public async Task Close_is_idempotent() { /* SweepAsync duas vezes → um bônus só */ }
[Fact] public async Task Complement_for_someone_elses_checkin_is_404() { }
```

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar. Complemento e fechamento travam a linha do `CheckIn` com `FOR UPDATE`, o que garante o limite de 15 sob concorrência.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(checkins): complements, closing and hydration bonus`.

**Marco:** com as Tasks 1–10 prontas, o plano 3 (mobile) pode começar.

---

### Task 11: Ligas: criar, convidar, aceitar, sair

**Files:**
- Create: `Data/Entities/League.cs`, `LeagueMembership.cs`, `LeagueInvite.cs`, uma migration
- Create: `src/TwoDrink.Api/Features/Leagues/LeagueService.cs`, `LeagueEndpoints.cs`, `InviteEndpoints.cs`
- Test: `tests/TwoDrink.Api.Tests/LeagueTests.cs`

**Interfaces:**
- Consumes: `SeasonService`, `BusinessTime`, `RulesOptions`.
- Produces:
  - `League`: `Guid Id`, `Guid SeasonId`, `Guid CreatorId`, `string Name` (1–60 caracteres), `string? Description` (até 280), `string? CoverBlobName`, `DateOnly StartDate`, `DateOnly EndDate`, `DateTimeOffset? TrophiesAwardedAt`.
  - `LeagueMembership`: `Guid LeagueId`, `Guid UserId`, `DateTimeOffset JoinedAt`, `DateTimeOffset? LeftAt`. A chave primária é `(LeagueId, UserId)`, o que impede voltar depois de sair.
  - `LeagueInvite`: `Guid Id`, `Guid LeagueId`, `string TokenHash`, `DateTimeOffset ExpiresAt`, `Guid? UsedBy`, `DateTimeOffset? UsedAt`.
  - Uma liga ocupa vaga enquanto `agora < fim+1 00:00` (fuso de São Paulo) e `LeftAt == null`.
  - Endpoints:
    - `POST /leagues` com `{ name, description, startDate, endDate }`. Valida que as datas cabem numa temporada (`dates_outside_season`), que `start ≤ end`, que `end ≥ hoje`, e a vaga (`league_limit`). O criador entra como membro.
    - `POST /leagues/{id}/invites`: só o criador (senão 403). Devolve `{ token, url: "https://2drink.app/i/{token}", expiresAt }`. O token tem 32 bytes aleatórios em base64url; o banco guarda só o hash.
    - `POST /invites/{token}/accept`. Valida o convite (`invite_invalid`, `invite_expired`, `invite_used`), a liga (`league_ended`), a saída anterior (`left_league`), se já é membro (`already_member`) e a vaga (`league_limit`).
    - `POST /leagues/{id}/leave`: o criador não sai de liga ativa (`creator_cannot_leave`).
    - `GET /me/leagues`: devolve `[{ id, name, startDate, endDate, status: upcoming|active|ended, memberCount }]`.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
[Fact] public async Task Create_league_outside_season_rejected() { /* dates_outside_season */ }
[Fact] public async Task Third_active_league_rejected() { /* cria 2 e depois a 3ª → league_limit */ }
[Fact] public async Task Ended_league_frees_slot() { /* avança o relógio além do fim → a 3ª é aceita */ }
[Fact] public async Task Only_creator_generates_invites() { /* membro → 403 */ }
[Fact] public async Task Invite_is_single_use() { /* B aceita; C usa o mesmo token → invite_used */ }
[Fact] public async Task Invite_expires_after_7_days() { /* Advance(7d + 1s) → invite_expired */ }
[Fact] public async Task Concurrent_accepts_only_one_wins() {
    var t = await NewInvite(); var rs = await Task.WhenAll(Accept(UserB, t), Accept(UserC, t));
    Assert.Single(rs, r => r.IsSuccessStatusCode);
    Assert.Single(rs, r => ProblemCode(r) == "invite_used");
}
[Fact] public async Task Accept_blocked_when_user_has_two_leagues() { /* league_limit */ }
[Fact] public async Task Leaving_is_permanent() { /* sai; novo convite → left_league */ }
[Fact] public async Task Creator_cannot_leave_active_league() { }
```

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar. O aceite usa `UPDATE league_invites SET used_by=…, used_at=… WHERE id=… AND used_by IS NULL` e checa se afetou 1 linha. A checagem de vaga fica na mesma transação, com advisory lock por usuário.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(leagues): create, single-use invites, accept, leave with slot limit`.

---

### Task 12: Ranking da liga e Troféu Dourado

**Files:**
- Create: `src/TwoDrink.Domain/Leagues/LeagueStandings.cs`
- Create: `src/TwoDrink.Api/Features/Leagues/StandingsQuery.cs`, `Jobs/TrophyAwarder.cs`, `Data/Entities/Trophy.cs`, uma migration
- Test: `tests/TwoDrink.Domain.Tests/LeagueStandingsTests.cs`, `tests/TwoDrink.Api.Tests/StandingsTests.cs`

**Interfaces:**
- Consumes: `LedgerEntry` (Task 5), `LeagueMembership` (Task 11), `BusinessTime.Range` (Task 4).
- Produces:
  - `record MemberScore(Guid UserId, decimal Points, int DistinctVenues)`
  - `static IReadOnlyList<(MemberScore Score, int Rank)> LeagueStandings.Rank(IEnumerable<MemberScore> s)`: ordena por pontos e depois por `DistinctVenues`, ambos decrescentes. Empate nos dois dá o mesmo `Rank`.
  - `static IReadOnlyList<Guid> LeagueStandings.Champions(IEnumerable<MemberScore> s)`: todos com `Rank == 1`. Lista vazia se o maior total for ≤ 0.
  - `StandingsQuery.ForLeagueAsync(Guid leagueId, CancellationToken ct)`. Para cada membro com `LeftAt == null`, considera os lançamentos da conta dele com `CreatedAt` em `[max(JoinedAt, início), fim)`:
    - `Points` = soma de todos os tipos, estornos incluídos;
    - `DistinctVenues` = quantidade de `VenueId` distintos entre os lançamentos `CheckIn`.
  - `GET /leagues/{id}/standings` (só membros): devolve `[{ userId, displayName, points, distinctVenues, rank }]`.
  - `Trophy`: `Guid Id`, `Guid LeagueId`, `Guid UserId`, `DateTimeOffset AwardedAt`. A chave única é `(LeagueId, UserId)`.
  - `TrophyAwarder.AwardDueAsync(CancellationToken ct)` (`BackgroundService` a cada 5 min): para ligas com `fim ≤ agora` e `TrophiesAwardedAt == null`, cria os troféus e marca a liga. É idempotente.

- [ ] **Step 1:** Escrever os testes que falham:

```csharp
// Domínio
[Fact] public void Tie_on_points_broken_by_distinct_venues() {
    var r = LeagueStandings.Rank([new(A, 10, 2), new(B, 10, 3)]);
    Assert.Equal(B, r[0].Score.UserId); Assert.Equal(2, r[1].Rank);
}
[Fact] public void Full_tie_gives_multiple_champions() =>
    Assert.Equal(2, LeagueStandings.Champions([new(A, 10, 3), new(B, 10, 3), new(C, 4, 1)]).Count);
[Fact] public void No_champion_when_nobody_scored() =>
    Assert.Empty(LeagueStandings.Champions([new(A, 0, 0)]));

// API
[Fact] public async Task Points_before_joining_do_not_count() { }
[Fact] public async Task Checkin_at_2330_on_last_day_counts_and_0001_next_day_does_not() {
    // Liga até 2027-02-15. Check-in em 2027-02-15T23:30-03:00 conta.
    // Check-in em outro bar em 2027-02-16T00:31-03:00 (respeitando 1h) não conta.
}
[Fact] public async Task Same_checkin_scores_in_both_leagues() { }
[Fact] public async Task Member_who_left_is_not_ranked() { }
[Fact] public async Task Reversal_reduces_points() { }
[Fact] public async Task Trophies_awarded_once_at_league_end() {
    Clock.SetUtcNow(DateTimeOffset.Parse("2027-02-16T03:00:01Z"));
    await Awarder.AwardDueAsync(default); await Awarder.AwardDueAsync(default);
    Assert.Equal(1, await TrophyCount(League));
}
[Fact] public async Task Standings_forbidden_for_non_members() { /* 403 */ }
```

- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(leagues): standings from ledger, tie-break and golden trophy`.

---

### Task 13: Renovar a liga (primeiro item a cortar se faltar tempo)

**Files:**
- Modify: `src/TwoDrink.Api/Features/Leagues/LeagueService.cs`, `LeagueEndpoints.cs`
- Test: `tests/TwoDrink.Api.Tests/LeagueRenewTests.cs`

**Interfaces:**
- Consumes: `LeagueService` (Task 11).
- Produces: `POST /leagues/{id}/renew` com `{ name?, startDate, endDate }`.
  - Só o criador, e só com a liga encerrada (`league_not_ended`).
  - Cria uma liga nova com os membros ativos da anterior. Quem não tem vaga fica de fora, e a resposta lista essas pessoas em `skippedUserIds`.
  - Copia a capa.

- [ ] **Step 1:** Escrever os testes que falham: `Renew_copies_active_members`, `Renew_skips_members_without_slot`, `Renew_requires_ended_league`, `Only_creator_renews`.
- [ ] **Step 2:** Rodar os testes. Esperado: FAIL.
- [ ] **Step 3:** Implementar.
- [ ] **Step 4:** Rodar os testes. Esperado: PASS.
- [ ] **Step 5:** Commit: `feat(leagues): renew ended league with same members`.

---

## Verificação final

- [ ] `dotnet test` na solution inteira: tudo verde.
- [ ] `dotnet build -warnaserror`: sem avisos.
- [ ] Subir localmente (`dotnet run --project src/TwoDrink.Api` com o Postgres do Docker) e fazer manualmente o fluxo: login fake → perfil → nearby → intenção → upload → confirmar → ligas → ranking.
