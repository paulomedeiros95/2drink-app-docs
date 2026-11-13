# Guia passo a passo: contas e serviços do 2DRINK (parte do dono do produto)

**Para quem:** você, dono do produto.
**Quando:** comece hoje. Estas etapas não travam a construção da API, mas travam a publicação nas lojas. O item mais urgente é a **Etapa 3 (Google Play e testadores)**.
**Tempo total:** cerca de 2 a 3 horas de trabalho seu, mais os dias de aprovação de cada empresa.

> Os nomes de botões e menus podem variar um pouco, porque Apple, Google e Microsoft mudam as telas com frequência. Se algo estiver diferente, procure pelo texto mais parecido ou me mande um print da tela (sem dados sensíveis).

---

## Regras de ouro (leia antes de tudo)

1. **Nunca cole no chat** chaves de API, senhas, códigos de verificação ou tokens. Nunca coloque essas informações em arquivos do repositório.
2. **Guarde tudo num gerenciador de senhas.** Se você não usa nenhum, instale o **Bitwarden** (gratuito; bitwarden.com). Crie uma pasta chamada **2DRINK** e, a cada etapa deste guia, salve ali o que estiver marcado com 🔐.
3. **Use um cartão de crédito internacional.** Todas as contas cobram em dólar ou pedem verificação de cartão. Um cartão virtual do seu banco serve.
4. **Ligue a verificação em duas etapas** em todas as contas criadas aqui.
5. Quando terminar cada etapa, marque o checkbox `[ ]` → `[x]` neste arquivo (ou só me avise no chat qual etapa terminou).

---

## Etapa 0: um e-mail exclusivo do projeto (15 min)

**Por quê:** as contas do Google Play e do Google Cloud ficam presas ao e-mail usado. Com um e-mail só do projeto, fica muito mais fácil transferir tudo para a empresa (CNPJ) depois, ou dar acesso a outra pessoa, sem misturar com o seu e-mail pessoal.

- [ ] 0.1. Abra **accounts.google.com/signup** num navegador.
- [ ] 0.2. Crie uma conta Gmail nova. Sugestões de endereço: `2drink.app@gmail.com`, `contato.2drink@gmail.com` ou parecido (o que estiver livre).
  - Nome: **2DRINK**. Sobrenome: pode deixar **App**.
  - Data de nascimento: a sua (precisa ser maior de 18).
- [ ] 0.3. Ative a verificação em duas etapas: entre em **myaccount.google.com → Segurança → Verificação em duas etapas → Ativar**. Use o seu celular.
- [ ] 0.4. 🔐 Salve no Bitwarden: e-mail e senha da conta.

> Daqui para frente, este guia chama esse endereço de **"e-mail do projeto"**.

---

## Etapa 1: Apple Developer, pessoa física (30 min + 1 a 2 dias de aprovação)

**Custo:** US$ 99 por ano (o valor em reais aparece na hora do pagamento).
**Por quê:** sem essa conta não dá para publicar na App Store nem ativar o "Entrar com Apple".

### 1.A. Preparar o Apple ID
- [ ] 1.1. Decida qual Apple ID usar. Recomendação: **criar um Apple ID novo com o e-mail do projeto**, em **account.apple.com** → "Criar Apple ID". Use o **seu nome verdadeiro** (é conta de pessoa física e a Apple confere com o documento).
- [ ] 1.2. Ative a autenticação de dois fatores nesse Apple ID. A Apple exige isso para desenvolvedores. No iPhone: **Ajustes → [seu nome] → Iniciar Sessão e Segurança → Autenticação de Dois Fatores**. Se o Apple ID novo não for o do seu iPhone, ative pelo site account.apple.com, na seção de segurança.
- [ ] 1.3. 🔐 Salve no Bitwarden: Apple ID e senha.

### 1.B. Inscrever-se no programa
O caminho mais fácil no Brasil é pelo app, porque ele faz a verificação de documento pelo celular.

- [ ] 1.4. No iPhone, instale o app **Apple Developer** (gratuito, na App Store).
- [ ] 1.5. Abra o app, toque em **Account** (Conta) e entre com o Apple ID do passo 1.1.
- [ ] 1.6. Toque em **Enroll Now** (Inscrever-se agora).
- [ ] 1.7. Quando perguntar o tipo de entidade, escolha **Individual / Sole Proprietor** (Pessoa física).
- [ ] 1.8. Preencha os dados exatamente como no seu documento: nome completo, endereço, telefone.
- [ ] 1.9. Siga a verificação de identidade: fotografe seu RG ou CNH e tire a selfie que o app pedir.
- [ ] 1.10. Aceite o contrato e pague com o cartão internacional.
- [ ] 1.11. Aguarde o e-mail **"Welcome to the Apple Developer Program"**. Costuma levar de 24 a 48 horas.

**Sem iPhone?** Faça pelo site: **developer.apple.com/programs/enroll → Start Your Enrollment**, entre com o Apple ID e siga os mesmos passos (1.7 a 1.11). A verificação de identidade pode levar mais tempo pelo site.

- [ ] 1.12. Quando o e-mail de boas-vindas chegar, me avise. **Não precisa configurar mais nada** agora: o identificador do app e o "Entrar com Apple" fazemos juntos no plano do app mobile.

**Se der problema:** se a Apple pedir mais documentos ou recusar, me mande o texto da mensagem (sem números de documento).

---

## Etapa 2: Azure (20 min)

**Custo agora:** zero. A conta gratuita dá crédito no primeiro mês. Os servidores só serão criados perto da publicação.
**Por quê:** é onde vão rodar a API, o banco de dados e as fotos.

### 2.A. Criar a conta
- [ ] 2.1. Abra **azure.microsoft.com/free** e clique em **Experimente gratuitamente** (ou "Start free").
- [ ] 2.2. Entre com uma **conta Microsoft**. Se não tiver uma, clique em "Criar uma" e use o **e-mail do projeto**.
- [ ] 2.3. Preencha o perfil: país **Brasil**, nome, telefone (vai chegar um SMS de verificação).
- [ ] 2.4. Informe o cartão. A Microsoft faz só uma verificação e **não cobra** enquanto você estiver na conta gratuita.
- [ ] 2.5. Aceite os termos e conclua. Você cai no **Portal do Azure** (portal.azure.com).
- [ ] 2.6. 🔐 Salve no Bitwarden: o e-mail e a senha da conta Microsoft.

### 2.B. Criar o alerta de gastos (proteção contra surpresas)
- [ ] 2.7. No portal, digite **"Orçamentos"** (ou "Budgets") na barra de busca do topo e abra **Gerenciamento de Custos | Orçamentos**.
- [ ] 2.8. Confira se o **Escopo** mostra a sua assinatura (algo como "Azure subscription 1"). Se não mostrar, clique em "Escopo" e escolha a assinatura.
- [ ] 2.9. Clique em **+ Adicionar**.
- [ ] 2.10. Preencha:
  - **Nome:** `2drink-mensal`
  - **Período de redefinição:** Mensal
  - **Data de expiração:** deixe a sugerida ou coloque daqui a 2 anos
  - **Valor:** `200` (na moeda da sua conta; ajuste se preferir outro teto)
- [ ] 2.11. Clique em **Avançar**. Em **Condições de alerta**, crie duas linhas:
  - Tipo **Real**, **80%**
  - Tipo **Real**, **100%**
- [ ] 2.12. Em **Destinatários de alerta (email)**, coloque o seu e-mail pessoal e o e-mail do projeto.
- [ ] 2.13. Clique em **Criar**.

> Importante: o orçamento **avisa**, mas **não desliga** nada sozinho. Se chegar um alerta, me avise.

- [ ] 2.14. **Pare aqui.** Não crie nenhum recurso (servidor, banco, armazenamento). Eu passo esse passo a passo quando chegar a hora.

---

## Etapa 3: Google Play Console, conta pessoal + testadores (40 min + alguns dias) ⚠️ URGENTE

**Custo:** US$ 25, pagamento único.
**Por quê:** é a única forma de publicar no Android. Como a conta é pessoal e nova, o Google exige um **teste fechado com pelo menos 12 testadores, inscritos por 14 dias seguidos**, antes de liberar a publicação. Esse teste precisa começar **até meados de dezembro**. Os testadores precisam ser recrutados **agora**.

### 3.A. Criar a conta
- [ ] 3.1. Abra **play.google.com/console/signup** e entre com o **e-mail do projeto**.
- [ ] 3.2. Quando perguntar quem vai usar a conta, escolha **"Para você mesmo"** (conta pessoal).
- [ ] 3.3. **Nome do desenvolvedor:** `2DRINK`. É o nome que aparece na loja, abaixo do nome do app.
- [ ] 3.4. Responda o questionário sobre sua experiência com Android e quantos apps pretende publicar. Responda com sinceridade; não afeta a aprovação.
- [ ] 3.5. Preencha os **dados de contato**: e-mail de contato (o do projeto) e telefone. O Google envia códigos para confirmar os dois.
- [ ] 3.6. Pague os US$ 25 com o cartão internacional.
- [ ] 3.7. Faça a **verificação de identidade**: nome legal completo, endereço e foto do RG ou CNH. Leva de alguns dias até uma semana.
- [ ] 3.8. O Google pode pedir para você **confirmar que tem acesso a um celular Android**: instale o app **Google Play Console** num Android, entre com o e-mail do projeto e siga as instruções. Se você não tem Android, use o de alguém de confiança só para esse passo.
- [ ] 3.9. 🔐 Salve no Bitwarden: "Google Play Console — conta pessoal, e-mail do projeto".
- [ ] 3.10. Quando a verificação for aprovada, me avise. **Não crie o app no console ainda**: fazemos juntos no plano do app mobile.

### 3.B. Recrutar os testadores (comece hoje)
- [ ] 3.11. Crie uma planilha (Google Planilhas serve) chamada **"2DRINK — testadores"** com estas colunas:

  | Nome | E-mail Google usado no Play Store do celular | Modelo do celular | Confirmou? |
  |---|---|---|---|

- [ ] 3.12. Liste de **15 a 20 pessoas**. O mínimo exigido é 12; os extras cobrem quem desinstalar ou desistir.
  - Precisam ter **celular Android** (iPhone não conta para essa regra).
  - Precisam ter **18 anos ou mais**.
  - O e-mail tem que ser **exatamente o da conta Google logada no Play Store do celular** deles. Peça para conferirem em: Play Store → foto do perfil no canto → o e-mail aparece ali.
- [ ] 3.13. Mande para cada um a mensagem abaixo (copie e ajuste):

  > Oi! Estou lançando um app chamado **2DRINK**, uma liga entre amigos para ir a bares e restaurantes, com pontos, badges e ranking (e que premia quem bebe água 💧). Preciso de testadores Android para a fase de teste do Google.
  >
  > O que você precisa fazer:
  > 1. Me mandar o **e-mail da conta Google que está no Play Store do seu celular** (abre o Play Store, toca na sua foto no canto: o e-mail aparece ali).
  > 2. Em **meados de dezembro** vou te mandar um link. Você aceita o convite e instala o app.
  > 3. **Deixa o app instalado por pelo menos 14 dias seguidos**, sem desinstalar. Se puder usar e me dizer o que achou, melhor ainda!
  >
  > Topa?

- [ ] 3.14. Conforme as respostas chegarem, preencha a planilha e marque **"Confirmou?"**.
- [ ] 3.15. Meta: **15 confirmados até 1º de dezembro**. Me avise quando chegar lá.

> Essa planilha tem e-mails de terceiros. **Não compartilhe publicamente** e não cole a lista no chat; quando chegar a hora, eu te digo onde colar os e-mails dentro do Play Console.

---

## Etapa 4: Google Cloud: busca de bares e login com Google (45 min)

**Custo:** a Places API cobra por busca, mas há uma faixa mensal gratuita e crédito inicial. Nos passos abaixo você coloca um **teto diário**, para o gasto nunca fugir do controle.

### 4.A. Criar o projeto
- [ ] 4.1. Abra **console.cloud.google.com** e entre com o **e-mail do projeto**. Aceite os termos de serviço.
- [ ] 4.2. No topo da tela, clique no **seletor de projetos** (ao lado do logo "Google Cloud") → **Novo projeto**.
- [ ] 4.3. **Nome do projeto:** `2DRINK`. **Local:** "Nenhuma organização". Clique em **Criar**.
- [ ] 4.4. Espere a notificação e **selecione o projeto 2DRINK** no seletor do topo. Confira que o nome aparece lá antes de seguir.

### 4.B. Faturamento e alerta de gastos
- [ ] 4.5. Menu ☰ (canto superior esquerdo) → **Faturamento**.
- [ ] 4.6. Se pedir para vincular uma conta de faturamento, clique em **Vincular uma conta de faturamento** → **Criar conta de faturamento**. País **Brasil**, tipo de conta **Individual**, cartão internacional. Se aparecer a oferta de **teste gratuito com crédito**, aceite.
- [ ] 4.7. Ainda em **Faturamento**, no menu lateral, clique em **Orçamentos e alertas** → **Criar orçamento**.
  - **Nome:** `2drink-mensal`
  - **Projetos:** marque só o **2DRINK**
  - **Valor:** `100` (na moeda da conta)
  - **Limites de alerta:** 50%, 90% e 100% (tipo "Real")
  - Marque **"Enviar alertas por e-mail para administradores e usuários de faturamento"**
  - Clique em **Concluir**.

### 4.C. Ativar a Places API (New)
- [ ] 4.8. Menu ☰ → **APIs e serviços** → **Biblioteca**.
- [ ] 4.9. Busque **"Places API (New)"**. Atenção: existe uma "Places API" antiga, **sem** "(New)". Use **a que tem "(New)" no nome**.
- [ ] 4.10. Abra e clique em **Ativar**.

### 4.D. Criar e restringir a chave de API
- [ ] 4.11. Menu ☰ → **APIs e serviços** → **Credenciais**.
- [ ] 4.12. Clique em **+ Criar credenciais** → **Chave de API**. A chave aparece numa janela.
- [ ] 4.13. 🔐 **Copie a chave e salve no Bitwarden** como "Google Places API key". **Não cole no chat.**
- [ ] 4.14. Na janela, clique em **Editar chave de API** (ou clique no nome da chave na lista).
- [ ] 4.15. **Nome:** `2drink-api-places`.
- [ ] 4.16. Em **Restrições de aplicativo**, deixe **Nenhuma** por enquanto. Quando a API estiver no Azure, restringimos pelo IP do servidor.
- [ ] 4.17. Em **Restrições de API**, marque **Restringir chave** e, na lista, selecione **somente "Places API (New)"**.
- [ ] 4.18. Clique em **Salvar**.

### 4.E. Teto diário de uso (proteção extra)
- [ ] 4.19. Menu ☰ → **APIs e serviços** → **APIs e serviços ativados** → clique em **Places API (New)** → aba **Cotas e limites do sistema** (ou "Quotas").
- [ ] 4.20. Procure a cota **"SearchNearby requests per day"** (ou parecida, com "SearchNearby" e "per day"). Marque a caixa da linha e clique em **Editar** (ícone de lápis).
- [ ] 4.21. Coloque **1000** e salve. Com o cache da API, 1.000 buscas por dia sobram para o teste e o Carnaval; se precisar, aumentamos.
- [ ] 4.22. Faça o mesmo para **"GetPlace requests per day"** (detalhes de um lugar): **1000**.

### 4.F. Tela de consentimento do login com Google
- [ ] 4.23. Menu ☰ → **APIs e serviços** → **Tela de consentimento OAuth**. (Em algumas contas o menu se chama **Google Auth Platform**, com as abas "Branding", "Público" e "Clientes".)
- [ ] 4.24. Clique em **Começar** (ou "Configurar").
- [ ] 4.25. **Informações do app:**
  - **Nome do app:** `2DRINK`
  - **E-mail para suporte do usuário:** o e-mail do projeto
- [ ] 4.26. **Público:** **Externo**.
- [ ] 4.27. **Dados de contato:** o e-mail do projeto e o seu pessoal.
- [ ] 4.28. Aceite a política de dados do usuário e clique em **Criar**.
- [ ] 4.29. Na aba **Público** (ou "Usuários de teste"), em **Usuários de teste**, clique em **+ Adicionar usuários** e adicione **o seu e-mail pessoal** e o **e-mail do projeto**. Enquanto o app estiver em "Teste", só esses e-mails conseguem entrar com Google. Os testadores da Etapa 3 entram nessa lista mais tarde.
- [ ] 4.30. Não mexa em **Escopos**: o padrão (nome, e-mail e foto) é o que o app usa.

### 4.G. Client ID do tipo Web
- [ ] 4.31. Vá em **APIs e serviços → Credenciais** (ou na aba **Clientes** do Google Auth Platform) → **+ Criar credenciais** → **ID do cliente OAuth**.
- [ ] 4.32. **Tipo de aplicativo:** **Aplicativo da Web**.
- [ ] 4.33. **Nome:** `2drink-web`.
- [ ] 4.34. Deixe **"Origens JavaScript autorizadas"** e **"URIs de redirecionamento"** **vazios**.
- [ ] 4.35. Clique em **Criar**.
- [ ] 4.36. 🔐 Salve no Bitwarden o **ID do cliente** (termina em `.apps.googleusercontent.com`). A **chave secreta do cliente** também aparece: salve, mas ela **não** será usada agora.
- [ ] 4.37. Os Client IDs de **Android** e **iOS** ficam para o plano do app mobile, porque precisam de dados do app que ainda não existem.

---

## Etapa 5: GitHub (5 min)

- [ ] 5.1. Confira que você consegue abrir os três repositórios logado na sua conta:
  - github.com/paulomedeiros95/2drink-app-docs
  - github.com/paulomedeiros95/2drink-api
  - github.com/paulomedeiros95/2drink-mobile
- [ ] 5.2. Se algum estiver **público**, decida se quer deixá-lo **privado**: em cada repo, **Settings → General → Danger Zone → Change visibility → Make private**. Recomendo privado para os três até o lançamento.
- [ ] 5.3. (Opcional) A integração do GitHub com o Claude Code falhou nesta sessão por um problema de autorização. Para reconectar, rode `/mcp` numa sessão interativa do Claude Code e autentique o servidor do GitHub. Não é obrigatório: o `git` normal funciona.

---

## Etapa 6: Domínio (opcional agora, 15 min)

Os links de convite usam um endereço provisório. Quando quiser o definitivo:

- [ ] 6.1. Escolha o domínio (por exemplo, `2drink.com.br` no **registro.br**, ou um `.app` / `.com` em outro registrador).
- [ ] 6.2. Registre no seu nome (CPF; depois dá para transferir para o CNPJ).
- [ ] 6.3. Me diga qual domínio registrou. Eu troco a configuração `Invites:BaseUrl`.

---

## Resumo do que guardar no Bitwarden (pasta 2DRINK)

| Item | Etapa |
|---|---|
| E-mail do projeto + senha | 0 |
| Apple ID + senha | 1 |
| Conta Microsoft (Azure) + senha | 2 |
| Google Play Console (é a mesma conta do e-mail do projeto) | 3 |
| Google Places API key | 4.D |
| Google OAuth Client ID Web (+ chave secreta) | 4.G |

**Como as chaves vão chegar à API:** quando a API precisar delas (perto da publicação), eu te passo os comandos para **você mesmo** rodar no seu computador, no formato `dotnet user-secrets set "Google:PlacesApiKey" "COLE-AQUI"`. A chave fica guardada fora do repositório, e você nunca precisa mandá-la para mim.

---

## Checklist final para me avisar

- [ ] Etapa 0: e-mail do projeto criado
- [ ] Etapa 1: e-mail de boas-vindas da Apple recebido
- [ ] Etapa 2: Azure criado, com orçamento
- [ ] Etapa 3: Google Play verificado + **15 testadores confirmados até 1º/12**
- [ ] Etapa 4: projeto Google Cloud com Places API (New), chave restrita, tetos diários, tela de consentimento e Client ID Web
- [ ] Etapa 5: repositórios conferidos
- [ ] Etapa 6 (opcional): domínio registrado
