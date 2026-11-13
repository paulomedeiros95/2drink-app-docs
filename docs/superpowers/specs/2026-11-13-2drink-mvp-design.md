# 2DRINK — Design do MVP (Carnaval 2027)

**Data:** 2026-11-13
**Status:** aprovado em 2026-11-13 (revisado na mesma data)
**Repositórios:** `2drink-app-docs` (este, fonte da verdade), `2drink-api` (API + painel admin), `2drink-mobile` (app)

Convenção deste documento: itens marcados **[decisão do Claude]** foram propostos durante o brainstorm e aceitos pelo dono do produto, ou são escolhas técnicas deixadas ao construtor. Todo o resto foi definido pelo dono do produto.

---

## 1. Intenção e o porquê

O 2DRINK transforma ir ao bar ou restaurante com os amigos num jogo social: o "Gymrats dos bares". Cada pessoa registra suas visitas (check-in com selfie), ganha pontos e badges colecionáveis, compete com os amigos em **ligas privadas** e disputa o **Troféu Dourado** de cada liga.

Clima do app: **amigável, fácil, simples e divertido**. Competição entre amigos, senso de comunidade, "grupo de responsabilidade". Frase-guia: *"Registre e compartilhe atividades com seus amigos."* O app incentiva **hidratação** (beber água) e nunca o consumo excessivo.

**Negócio:** gratuito para o usuário (B2C). A receita vem dos bares (B2B2C), que compram um **tier de badge** e passam a atrair grupos que querem "farmar pontos". O MVP precisa provar que um grupo muda de bar por causa de um badge de tier alto.

**Lançamento:** Carnaval 2027 (6 a 10 de fevereiro de 2027). Isso deixa cerca de 12 semanas a partir desta data, incluindo a revisão nas lojas da Apple e do Google.

## 2. Objetivos, não-objetivos e anti-objetivos

### Objetivos
- Core loop completo: entrar numa liga → ir a um bar → fazer check-in com selfie → lançar bebidas → ganhar pontos e badges → interagir no feed → disputar o troféu.
- Bares parceiros com tier e arte de badge personalizados, configurados por um painel admin seguro.
- Pontuação à prova de manipulação: um extrato imutável.
- Aprovação nas lojas da Apple e do Google a tempo do Carnaval.
- Medir os KPIs da seção 10.

### Não-objetivos (fora do MVP)
- Qualquer coisa pública ou global: ranking público, perfis públicos, seguir pessoas fora das ligas, busca de usuários.
- Chat privado.
- Versão paga (por exemplo, até 10 ligas). O limite só precisa ser configurável.
- Feature Fitness (calorias e treinos). Entra só um teaser "em breve", sem funcionalidade.
- Verificação automática da selfie: o MVP usa o código de honra.
- Exibir patrocínio ou anúncio dentro do app além do próprio badge.
- Versão web para usuários finais.

### Anti-objetivos (seria um fracasso mesmo funcionando)
- Um app que premia ficar bêbado ou que soa como "vira tudo". Isso também causa rejeição nas lojas (Apple 1.4.3).
- Trolls ou desconhecidos estragando a credibilidade do jogo.
- Pontos que possam ser alterados por usuário, desenvolvedor ou invasor sem deixar rastro.
- Alguém sem autorização colocando um bar em tier alto pelo painel.

## 3. Restrições

- **Contas:** ainda não existem as contas Apple Developer, Google Play Console e Azure. **Decisão: começar com contas de pessoa física** e migrar para empresa (CNPJ) depois. A Apple permite migrar a conta; no Google a migração é a transferência do app para uma conta de empresa. Criar as contas é o primeiro item do plano (ver 9).
- **Prazo:** submissão às lojas no início de janeiro de 2027, para ter margem de revisão e correções antes de 6 de fevereiro.
- **Lojas:**
  - Idade mínima de 18 anos.
  - Conteúdo gerado por usuário exige denunciar, bloquear e moderar (Apple 1.2).
  - Login com Apple é obrigatório quando o app oferece login com Google.
  - A exclusão de conta tem que ser possível dentro do app.
  - Nenhuma linguagem de incentivo ao consumo excessivo.
- **Brasil:**
  - LGPD: consentimento, exclusão de dados e fotos privadas.
  - Atenção à Lei 9.294/96 e ao CONAR na comunicação sobre bebida alcoólica.
- **Google Places:** os termos de uso permitem guardar o `place_id` indefinidamente. Os demais dados (nome, coordenadas, endereço) só podem ficar em cache por tempo limitado (30 dias para coordenadas) e precisam ser renovados.
- **Localização:** apenas "durante o uso do app". Sem localização em segundo plano.
- **Regras sempre no servidor:** o app só exibe e envia. Pontos, badges, limites e validações são calculados e aplicados pela API.

## 4. Abordagens consideradas

| | Abordagem | Ganha | Perde |
|---|---|---|---|
| **A (escolhida)** | API ASP.NET Core (.NET 10) + PostgreSQL; app React Native com Expo; painel admin em Blazor no repo da API; hospedagem Azure (incluindo Azure Blob para fotos e artes) | Caminho mais curto e confiável até as lojas: câmera, GPS, notificações e builds via Expo/EAS estão maduros. Mantém o .NET já sinalizado no repositório | Duas linguagens (C# e TypeScript) |
| B | .NET + Flutter | Interface e animações muito fluidas | Dart é menos comum; publicação um pouco mais trabalhosa |
| C | .NET + .NET MAUI | Uma linguagem só | Componentes de câmera, mapa e notificação menos maduros; maior risco para o prazo |
| Descartada | Backend pronto (Firebase/Supabase) | Rapidez inicial | Regras espalhadas no cliente; extrato imutável e painel seguro frágeis |

**Escolha do dono do produto: A**, por ser o caminho mais curto até o Carnaval mantendo .NET. O Azure Blob é usado para fotos e artes de badge.

## 5. Regras do produto

### 5.1 Conta e cadastro
- Login com **Google** e com **Apple**.
- **Data de nascimento obrigatória.** Só entra quem tem 18 anos ou mais.
- No cadastro, a pessoa aceita os **Termos de Uso**, a **Política de Privacidade** e o **Código da Liga**. O Código da Liga explica a regra de honra da selfie e traz "beba com responsabilidade, se beber não dirija".
- Perfil: nome de exibição e foto de perfil (pode vir da galeria).
- **Exclusão de conta dentro do app** (ver 5.10).
- **Idiomas:** português e inglês desde o MVP. Um botão com a bandeira permite trocar o idioma a qualquer momento. O idioma inicial segue o do aparelho **[decisão do Claude]**.
- **Textos legais:** os Termos de Uso, a Política de Privacidade e o Código da Liga são redigidos pelo Claude durante o desenvolvimento, em PT e EN, com base na LGPD, no Marco Civil da Internet, no CDC e na Lei 9.294/96. **Precisam ser revisados por um advogado antes da publicação.** Os dados da empresa (razão social, CNPJ, contato do encarregado de dados) entram como campos a preencher.

### 5.2 Bares (estabelecimentos)
- O público-alvo são **bares e restaurantes**.
- A lista vem do **Google Places**: na tela de check-in, o app mostra os estabelecimentos próximos do aparelho. A API faz a consulta ao Google (a chave nunca fica no app) e reaproveita resultados recentes da mesma região para controlar o custo **[decisão do Claude]**.
- O banco do 2DRINK guarda o `place_id` e, ligados a ele, os dados próprios (tier, arte, histórico de patrocínio). Nome e coordenadas ficam em cache e são renovados dentro do prazo permitido pelo Google.
- Todo estabelecimento começa no **tier Default**, com a **arte padrão** de badge.
- Para os **bares parceiros**, o admin define o tier e sobe uma arte própria pelo painel.

**Tiers e pontos do badge:**

| Tier | Pontos do badge |
|---|---|
| Default | 1 |
| Bronze | 3 |
| Prata | 5 |
| Ouro | 10 |
| Diamante | 25 |
| Epic | 50 |

- **O patrocínio tem prazo** [decisão do Claude]: cada tier pago é cadastrado com data de início e de fim. No fim do prazo o bar volta automaticamente para Default (e para a arte padrão, salvo outra configuração do admin).

### 5.3 Check-in (foto + bebidas, dentro do raio)
**Jornada típica:** a pessoa chega ao bar e bebe (por exemplo, 5 águas). Antes de ir embora, ainda no raio de 70m, ela abre o app, tira a selfie, informa o que bebeu e quanto, e envia. Esse envio é o check-in.

- O check-in é **um envio só**, com:
  - a **selfie tirada na hora pela câmera do app** (a pessoa dentro do estabelecimento, segurando a bebida). **A galeria é proibida.** O app deixa claro o que a selfie precisa mostrar;
  - a **lista de bebidas** (tipo e quantidade), com **pelo menos 1 e no máximo 15**.
- **A foto não é verificada automaticamente.** Vale o código de honra da liga.
- **Presença física:** a posição do aparelho precisa estar no **raio de 70m** do estabelecimento no momento do envio. A API compara a posição enviada com as coordenadas do bar e aceita quando `distância ≤ 70m + margem de precisão`. A margem é a precisão que o próprio celular informa, limitada a um teto (sugestão: 50m) **[decisão do Claude]**. Fora do raio, o check-in é recusado e a pessoa pode tentar de novo.
- **Localização falsa:** no Android, a API rejeita posições marcadas pelo sistema como simuladas ("mock"). No iOS não existe sinal equivalente confiável, então vale o código de honra. Ninguém consegue bloquear 100%.
- **Pontuação do check-in:**
  - **cada bebida pontua**, em toda visita (tabela abaixo);
  - na **primeira visita da temporada**, ou quando o tier atual do bar é maior que o maior badge que a pessoa já tem dele nesta temporada, o check-in também **desbloqueia o badge** e soma os pontos dele (5.6);
  - nas outras visitas não há badge novo, só os pontos das bebidas.
- **Lançamentos no extrato:** um "check-in" com 0 ponto (guarda foto e local), lançamentos "bebida" com os pontos de cada bebida e, se for o caso, um lançamento "badge" **[decisão do Claude]**.
- **Intervalos:**
  - **Mesmo bar:** 15h desde o último check-in da pessoa naquele bar.
  - **Bares diferentes:** 1h desde o último check-in da pessoa em qualquer bar.
- Os intervalos são configuráveis no painel **[decisão do Claude]**.

### 5.4 Complementar bebidas depois do check-in
- Quem prefere fazer o check-in na chegada pode **acrescentar bebidas** ao mesmo check-in depois, **sem foto nova**, enquanto estiver no raio (mesma regra de 70m + margem) **[decisão do Claude]**. Assim as duas formas de usar continuam possíveis: lançar tudo no fim ou aos poucos.
- O **limite de 15 bebidas** vale para o check-in inteiro, somando o envio inicial e os complementos.
- Uma tentativa fora do raio só é recusada. O check-in continua aberto para complementos **[decisão do Claude, após a revisão]**.
- O check-in deixa de aceitar complementos quando acontece a primeira destas situações **[decisão do Claude]**:
  1. a pessoa faz check-in em outro bar;
  2. passam 12h desde o check-in.

**Tipos de bebida e pontos:**

| Categoria | Exemplos | Pontos por unidade |
|---|---|---|
| Alcoólica | cerveja, drink, gim, shot, vinho | 1 |
| Não alcoólica | água, suco, refrigerante, whey protein | 1,5 |

O dono do produto definiu 1,5 para água; estender a suco, refrigerante e whey foi proposta do Claude. A lista de tipos e os pontos são configuráveis no painel **[decisão do Claude]**.

### 5.5 Meta de hidratação
- **Meta por check-in:** pelo menos 1 água para cada 2 bebidas alcoólicas, arredondando para cima, e no mínimo 1 água. Em fórmula: `águas ≥ max(1, ⌈alcoólicas / 2⌉)` **[decisão do Claude]**.
- Quem bate a meta ganha o selo **"Hidratado 💧"** no post e **+2 pontos bônus**.
- **O bônus entra no extrato na hora em que a meta é batida** (no check-in ou num complemento). Se um complemento posterior desfizer a meta (mais bebidas alcoólicas sem água), entra um estorno de −2; se ela for batida de novo, entra outro +2. Assim o ranking está sempre certo no momento em que a liga termina **[decisão do Claude, corrigida na revisão: antes o bônus só entrava 12h depois, o que podia mudar o ranking depois do troféu entregue]**.
- Só **água** conta para a meta. Suco, refrigerante e whey pontuam 1,5, mas não contam **[decisão do Claude]**.

### 5.6 Badges
Os badges são **por pessoa, por bar e por temporada**. Ao fazer check-in num bar, a pessoa ganha o badge do tier atual (e os pontos dele) quando:
- não tem nenhum badge desse bar nesta temporada; **ou**
- o tier atual do bar é **maior** que o maior tier de badge que ela já tem desse bar nesta temporada (**evolução**).

Regras complementares:
- **Tier que caiu não dá badge** [decisão do Claude].
- **Histórico protegido:** os pontos do badge são fixados no momento do check-in. Mudanças legítimas de tier depois disso não alteram nada.
- Cada pessoa acumula uma **lista de badges por bar** (por exemplo, Bronze e depois Ouro).
- **Dentro da mesma temporada, renovação não dá badge repetido nem menor.** Exemplo: o Higor ganhou Epic; o patrocínio acabou e depois o bar contratou Ouro (ou Epic de novo) na mesma temporada. O Higor não ganha nada novo ali, só os pontos das bebidas. Quem ainda não tinha badge daquele nível ganha normalmente.
- **Sala de Honra:** quando a temporada acaba, todos os badges dela vão para a "Sala de Honra" da pessoa, uma sala de troféus de badges, separada por temporada. Na temporada seguinte tudo reinicia, e a primeira visita a cada bar volta a valer badge.
- Cada badge guarda a arte vigente no momento da conquista [decisão do Claude].

### 5.7 Temporadas e ligas
**Temporadas**
- O **admin cria as temporadas**, duas por ano (por exemplo, "2027 – 1º semestre" e "2027 – 2º semestre"). As temporadas não se sobrepõem.

**Ligas**
- A **liga** é o desafio, o grupo de responsabilidade. Quem cria escolhe **nome, descrição, foto de capa** (pode vir da galeria, porque não é prova, e pode ser denunciada) e **data de início e de fim**, que precisam caber dentro da temporada.
- **Convites:**
  - só o **criador** gera links de convite;
  - cada link vale para **uma pessoa** (uso único);
  - o link expira [decisão do Claude: 7 dias].
- **Limite de ligas por pessoa: 2 ligas** no MVP, configurável no painel (a versão paga futura terá 10). Contam as ligas **ativas e as que ainda vão começar**; ligas encerradas não ocupam vaga. Criar uma liga ocupa uma vaga. Quem já está em 2 ligas precisa sair de uma antes de aceitar um convite.
- **Sair de uma liga é definitivo naquela liga**: a pessoa não pode voltar a ela [decisão do Claude].
- **O criador não pode sair da própria liga enquanto ela não terminar**, porque só ele gera convites [decisão do Claude].
- **Renovar:** quando a liga termina, o criador pode abrir uma nova liga com os mesmos membros e novas datas, sem novos convites [decisão do Claude]. Cada membro precisa ter vaga livre.

**Pontuação da liga**
- Os pontos da pessoa numa liga são a soma dos lançamentos do extrato dela (bebidas, badges, bônus de hidratação e estornos) feitos **depois de entrar na liga** e **dentro do período da liga**.
- **Estornos seguem o lançamento original:** um estorno só conta numa liga se o lançamento que ele estorna também contou nela, isto é, se o original foi feito dentro da janela daquela pessoa naquela liga. Exemplo: o bônus de hidratação foi ganho antes de a pessoa entrar na liga e desfeito depois; o estorno não desconta nada nessa liga (decisão do dono do produto, 2026-11-13).
- O mesmo check-in pontua em **todas** as ligas ativas da pessoa.

**Troféu Dourado**
- É entregue **quando a liga termina**, a quem tiver mais pontos.
- **Desempate:** mais estabelecimentos diferentes visitados (check-ins) dentro do período da liga.
- Se o empate persistir, **todos os empatados levam o troféu** [decisão do Claude].
- O troféu fica para sempre na galeria da pessoa.

**Alertas de fim**
- Notificação de "faltam 2 dias" para o fim de cada liga.

### 5.8 Feed
- **Um feed único por pessoa**, que junta os check-ins de todos que estão em alguma liga **ativa** com ela. Exemplo: Lucas (liga 1) e Pedro (liga 2) aparecem no mesmo feed.
- Nada é público. Quem não divide nenhuma liga com você nunca vê seus posts.
- **Cada post aparece uma vez só**, mesmo que você e a pessoa estejam juntos nas duas ligas.
- **Quais posts você vê:**
  - quem entra numa liga vê **todo o histórico de fotos da liga desde o início dela**, inclusive check-ins dos membros feitos antes de a pessoa entrar. Isso vale só para a visualização: a pontuação continua contando apenas a partir da entrada (5.7);
  - de cada liga ativa sua, você vê os check-ins dos membros feitos dentro do período daquela liga;
  - quando uma liga termina, ou você sai dela, os posts que só ela tornava visíveis saem do feed. O histórico da liga encerrada continua acessível dentro da própria liga, para quem era membro **[decisão do Claude]**;
  - a mesma regra vale para os links temporários das fotos.
- **Conteúdo do post:**
  - a selfie;
  - o nome do estabelecimento;
  - o badge ganho (se houver);
  - a contagem de bebidas, atualizada se houver complementos (por exemplo, "🍺 3 · 💧 2");
  - o selo "Hidratado 💧" (se a meta foi batida).
- **Interações:**
  - reações com um conjunto fixo de emojis;
  - comentários só de texto.
- **Filtro de palavrões e termos ilegais nos comentários:**
  - listas em português e em inglês, editáveis no painel;
  - normaliza acentos, troca de letras por números e símbolos ("p0rr@") e letras separadas;
  - o comentário bloqueado não é enviado, e a pessoa vê o motivo.
- **Moderação:**
  - denunciar post, comentário, capa de liga ou pessoa;
  - bloquear uma pessoa: os posts e comentários dela somem para quem bloqueou;
  - o autor e o criador da liga podem apagar posts;
  - as denúncias chegam ao painel admin.
- **Apagar o post esconde a foto e o post, mas não mexe no extrato.** Os pontos continuam valendo.

### 5.9 Outras telas e notificações
**Barra de navegação**

```
[ 🏠 Feed ]  [ 🏆 Ligas ]  [ 📸 Check-in ]  [ ✨ Fitness (em breve) ]  [ 👤 Perfil ]
```

**Telas**
- **Ligas:** lista das ligas da pessoa. Dentro de cada liga: capa, ranking (pontos e lugares visitados), datas e tempo restante, membros, e convites (só para o criador).
- **Perfil e galeria:**
  - badges da temporada atual;
  - **Sala de Honra**: os badges das temporadas anteriores, separados por temporada;
  - Troféus Dourados;
  - o próprio extrato de lançamentos, só para leitura.
- **Fitness (em breve):** ícone chamativo. Ao tocar, abre uma tela "Em breve" com um texto curto sobre calorias e treinos. Cada toque é registrado como evento para medir o interesse. Não tem nenhuma funcionalidade.

**Notificações push**
- alguém da sua liga fez check-in;
- alguém passou você no ranking;
- alguém reagiu ou comentou no seu post;
- faltam 2 dias para o fim da liga;
- você ganhou o Troféu Dourado.

**Tom dos textos**
- Zoeira de amigo ("o Higor tá farmando água 💧"), nunca "vira tudo".
- No check-in aparecem lembretes leves de responsabilidade, sem sermão.

### 5.10 Exclusão de conta (LGPD)
- Apaga os dados pessoais, as fotos (selfies e foto de perfil) e os comentários.
- A pessoa sai das ligas ativas e deixa de aparecer nos rankings delas.
- Os lançamentos do extrato ficam **anônimos**: o vínculo com a pessoa é trocado por um identificador sem dados pessoais. Assim a cadeia do extrato não quebra e os rankings já encerrados ficam consistentes.
- O campo "assinatura" do extrato (5.11) **não pode incluir dados pessoais**, para a anonimização não invalidar a cadeia.

### 5.11 Extrato imutável (pontuação à prova de manipulação)
- Todo evento que gera ou corrige pontos é um **lançamento**:
  - check-in;
  - bebida lançada;
  - badge concedido;
  - bônus de hidratação;
  - estorno.
- Cada lançamento guarda:
  - pessoa;
  - data e hora do servidor;
  - tipo;
  - estabelecimento;
  - check-in de origem;
  - pontos (o valor fixado naquele momento);
  - referência ao lançamento anterior.
- **Lançamentos nunca são alterados nem apagados.** A conta de banco de dados usada pela API só tem permissão de inserir e ler nessa tabela.
- **Correções entram como estorno:** um novo lançamento com pontos negativos, o motivo, o admin que fez e a referência ao lançamento original.
- **Cadeia de assinaturas:** cada lançamento tem um hash calculado sobre os próprios dados e o hash do lançamento anterior da mesma pessoa. Uma rotina periódica verifica as cadeias e alerta se alguma estiver quebrada.
- Os pontos de uma liga e o ranking são **sempre derivados do extrato**. Pode existir cache ou projeção para desempenho, mas a fonte da verdade é o extrato.
- O horário vem sempre do servidor, nunca do aparelho.

## 6. Painel de administração

**Onde fica e quem acessa**
- É um **site separado** do app: Blazor, dentro do repo `2drink-api`, como um projeto próprio, publicado num endereço próprio.
- Usa **contas nominais**. Não existe cadastro aberto: as contas são criadas por um procedimento controlado, fora do painel [decisão do Claude].
- **Login com segundo fator obrigatório** (código TOTP no celular).
- Sessão curta, bloqueio após tentativas falhas e, se possível, restrição por IP [decisão do Claude].

**Funções**
- **Temporadas:** criar e editar (as datas não se sobrepõem).
- **Bares parceiros:** buscar estabelecimentos do Google; definir tier com data de início e de fim; subir a arte do badge (Azure Blob); ver o histórico de patrocínio.
- **Anulação de badges:** selecionar badges concedidos num bar dentro de um intervalo de tempo e gerar estornos com motivo. Serve para o caso de uma mudança de tier fraudulenta.
- **Moderação:** fila de denúncias, ocultar conteúdo e suspender contas.
- **Configurações:**
  - limite de ligas por pessoa;
  - intervalos de check-in;
  - raio e teto de margem;
  - limite de bebidas por check-in;
  - tipos de bebida e pontos;
  - lista de palavras proibidas.
- **Registro de auditoria:** toda ação no painel guarda quem fez, quando, e o valor antes e depois. Esse registro também é só de inserção.

## 7. Arquitetura (abordagem A)

```
[ App Expo (iOS/Android) ] ──HTTPS──▶ [ API ASP.NET Core (.NET 10) ] ──▶ [ PostgreSQL (Azure) ]
                                          │        │
                                          │        ├──▶ [ Azure Blob: selfies, capas, artes ]
                                          │        ├──▶ [ Google Places API ]
                                          │        └──▶ [ Push: Expo Push → APNs/FCM ]
[ Painel Admin (Blazor, deploy separado) ] ┘ (mesmo banco; contas e permissões próprias)
```

- **App (`2drink-mobile`):**
  - Expo/React Native em TypeScript, com builds e envio às lojas via EAS.
  - Câmera dentro do app, sem seletor de galeria para selfie.
  - Localização só "durante o uso".
  - Notificações via Expo.
- **API (`2drink-api`):**
  - Toda regra de negócio fica aqui.
  - Autenticação com tokens próprios emitidos depois do login com Google ou Apple.
  - Limite de requisições por usuário.
- **Fotos:**
  - Container privado no Azure Blob.
  - Upload por link temporário de escrita, sem a foto passar pela API, emitido só depois que o check-in foi validado [decisão do Claude].
  - Leitura por links temporários, entregues só a quem pode ver o post.
  - Metadados da imagem (EXIF, incluindo GPS) removidos.
- **Docs:**
  - Este repositório é a fonte da verdade.
  - No início da implementação de cada repo, entra como *git submodule* em `docs/`.
  - Um `CLAUDE.md` em cada repo aponta para esta especificação.

**Ordem dos projetos**
1. API e banco (inclusive extrato e regras).
2. App mobile.
3. Painel admin.

O app e o painel dependem da API. O painel pode ser mínimo no início (tiers, temporadas, denúncias).

## 8. Segurança (resumo)

- As regras ficam no servidor. O app não é confiável.
- **Extrato:** só inserção, cadeia de hash e estornos auditados.
- **Painel admin:** endereço separado, segundo fator obrigatório, contas nominais sem cadastro aberto e auditoria de toda alteração.
- **Fotos:** privadas, links temporários e EXIF removido.
- **Segredos:** a chave do Google e as credenciais ficam só no servidor (Azure Key Vault ou equivalente) [decisão do Claude].
- **Abuso:** limite de requisições, intervalos de check-in e posição simulada rejeitada no Android.

## 9. Riscos conhecidos

- **Revisão das lojas:** o ponto por bebida alcoólica continua sensível. Mitigações: público 18+, mistura de bebidas, incentivo à hidratação, tom responsável e foco em bares e restaurantes. A aprovação não é garantida, por isso a submissão é antecipada.
- **GPS dentro de bares:** a margem de precisão ajuda, mas ainda pode haver bloqueios indevidos. Os valores são configuráveis para ajuste rápido.
- **Custo do Google Places:** cresce com o uso. O cache e a consulta só na tela de check-in contêm o custo.
- **Prazo de 12 semanas** para três entregas.
- **Google Play com conta pessoal:** exige teste fechado com 12 ou mais testadores, inscritos por 14 dias seguidos, antes de publicar. O teste precisa começar **até meados de dezembro de 2026**, com uma versão já funcional do app e os testadores recrutados antes. Esse é o caminho crítico do Android.
- **Corte, se faltar tempo** (nesta ordem): renovar liga → histórico de patrocínio no painel → restrição por IP no painel. A notificação "passou você no ranking" **não** sai.

## 10. KPIs e eventos a registrar

| KPI | Como medir |
|---|---|
| Retenção de fim de semana | % de usuários com check-in em dois fins de semana consecutivos |
| Engajamento de liga | Média de posts (check-ins com foto) por liga |
| Liquidez de badges (B2B) | Check-ins em bares Ouro, Diamante ou Epic, comparados com o histórico de visitas do grupo |
| Viralidade | Convites gerados → convites aceitos → contas novas |
| Interesse em Fitness | Toques no menu "em breve" por usuário ativo |

## 11. Deixado para o construtor

- Modelagem detalhada do banco, nomes de endpoints e formato dos tokens.
- Bibliotecas específicas (filtro de palavrões, hashing, cliente Google).
- Estratégia de cache dos resultados do Google e de projeção do ranking.
- Conjunto exato de emojis de reação e textos de interface (respeitando o tom da seção 5.9).
- Arte padrão de badge e identidade visual (a definir com o dono do produto em sessão de design).
- Estrutura de ambientes (desenvolvimento, homologação, produção) e pipeline de deploy.
