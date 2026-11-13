# Guarda-Roupa Digital + Marketplace — Casos de Uso e Telas do MVP

> Documento de apoio para implementação de um **MVP apenas em HTML/CSS/JS** (sem back-end), baseado no Business Model Canvas do projeto.

---

## 1. Visão geral

**Ideia:** aplicativo de guarda-roupa digital integrado a um marketplace de roupas usadas. O usuário cataloga suas peças, descobre o que usa pouco e pode anunciar/trocar essas peças.

**Proposta de valor (do Canvas):**
1. Facilitar o inventário de peças de roupas.
2. Possibilitar a troca ou venda de itens não utilizados.
3. Mostrar peças pouco utilizadas pelo usuário.

**Segmentos:** Pessoa física.

**Receita (do Canvas):** comissão por venda e promoção de anúncio (destaque).

### 1.1 Escopo do MVP

| Dentro do escopo | Fora do escopo (simulado ou futuro) |
|---|---|
| Cadastro/login simulado | Autenticação real |
| CRUD de peças | Back-end e banco de dados reais |
| Guarda-roupa com filtros | Pagamento real (será simulado) |
| Criar/gerenciar anúncios | Logística real / frete |
| Marketplace com busca e filtros | Chat em tempo real |
| Compra e proposta de troca (simuladas) | Venda de dados |
| Peças pouco usadas (alerta) | IA para sugestão de looks (versão simples/manual) |
| Destaque de anúncio (simulado) | Notificações push |

### 1.2 Premissas técnicas (MVP só HTML)

- **Persistência:** `localStorage` (limite ~5 MB). Estruturas em JSON.
- **Imagens:** converter para base64 e **redimensionar via `<canvas>`** (ex.: máx. 600 px) antes de salvar, para não estourar o limite.
- **Navegação:** SPA simples com *hash routing* (`#/guarda-roupa`) **ou** múltiplos arquivos `.html`. Recomendado: um `index.html` + roteamento por hash.
- **Dados iniciais (seed):** carregar ~15 anúncios fictícios de outros usuários para o marketplace ter conteúdo.
- **Responsivo:** *mobile-first* (o produto é um "aplicativo"). Largura de referência: 390 px.
- **Sem frameworks obrigatórios:** HTML + CSS + JS puro (opcional: Tailwind via CDN).

---

## 2. Atores

| Ator | Descrição |
|---|---|
| **Visitante** | Não logado. Só vê a tela de boas-vindas/login. |
| **Usuário** | Cataloga peças, anuncia, compra, propõe trocas. |
| **Sistema** | Calcula comissão, identifica peças pouco usadas, sugere looks. |

---

## 3. Modelo de dados (localStorage)

```json
{
  "usuario": {
    "id": "u1", "nome": "Maria", "email": "maria@email.com",
    "cidade": "Passo Fundo - RS"
  },
  "pecas": [{
    "id": "p1", "donoId": "u1", "nome": "Jaqueta jeans",
    "foto": "data:image/jpeg;base64,...",
    "categoria": "Casacos", "cor": "Azul", "tamanho": "M", "marca": "Levi's",
    "estacao": ["Outono", "Inverno"], "ocasiao": "Casual",
    "estado": "Ótimo", "usos": 2, "ultimoUso": "2026-03-10",
    "dataCadastro": "2026-01-05", "status": "no_armario | anunciada | vendida | trocada"
  }],
  "anuncios": [{
    "id": "a1", "pecaId": "p1", "vendedorId": "u1",
    "titulo": "Jaqueta jeans Levi's M", "descricao": "...",
    "tipo": "venda | troca | ambos", "preco": 120.00,
    "preferenciaTroca": "Prefiro camisas M", "destaque": false,
    "status": "ativo | pausado | vendido", "dataCriacao": "2026-04-01"
  }],
  "favoritos": ["a3", "a7"],
  "transacoes": [{ "id": "t1", "anuncioId": "a1", "compradorId": "u2", "valor": 120, "comissao": 12, "tipo": "venda" }],
  "propostasTroca": [{ "id": "pt1", "anuncioId": "a1", "pecaOferecidaId": "p9", "deId": "u2", "status": "pendente | aceita | recusada" }],
  "looks": [{ "id": "l1", "nome": "Look de trabalho", "pecasIds": ["p1", "p2", "p5"] }]
}
```

**Listas de valores fixos**
- **Categorias:** Camisetas, Camisas, Blusas, Calças, Shorts/Bermudas, Saias, Vestidos, Casacos, Calçados, Acessórios, Roupa íntima/Moda praia, Outros.
- **Estações:** Primavera, Verão, Outono, Inverno, Todas.
- **Tamanhos:** PP, P, M, G, GG, XG, Único, numerações 34–46 (calçados/calças).
- **Estado da peça:** Novo com etiqueta, Seminovo, Bom, Com detalhes.
- **Ocasião:** Casual, Trabalho, Festa, Esporte, Casa.

---

## 4. Regras de negócio

| ID | Regra |
|---|---|
| RN01 | Toda peça deve ter **nome, categoria, cor, tamanho e ao menos uma estação**. Foto é recomendada (se ausente, usar ícone padrão da categoria). |
| RN02 | Só é possível **anunciar peças do próprio guarda-roupa** com status `no_armario`. |
| RN03 | Uma peça só pode ter **um anúncio ativo** por vez. |
| RN04 | Ao criar o anúncio, a peça passa para `anunciada`. Ao pausar/excluir o anúncio, volta para `no_armario`. |
| RN05 | **Comissão** padrão de **10%** sobre vendas. Troca não tem comissão. |
| RN07 | Anúncio **destacado** aparece no topo do marketplace com etiqueta "Destaque" (custo simulado: R$ 4,90 por 7 dias). |
| RN08 | Peça **pouco utilizada** = `usos ≤ 2` **e** cadastrada há mais de 90 dias, **ou** sem uso há mais de 180 dias. |
| RN09 | Usuário **não pode comprar nem propor troca** no próprio anúncio. |
| RN10 | Anúncio do tipo `troca` não exibe preço; tipo `ambos` exibe preço e preferência de troca. |
| RN11 | Após venda concluída, a peça muda para `vendida` e **sai** do guarda-roupa ativo (vai ao histórico). |
| RN12 | Preço mínimo de anúncio: R$ 5,00. |
| RN13 | Ao aceitar uma troca, as duas peças trocam de dono (simulado) e os anúncios são encerrados. |

---

## 5. Mapa de navegação

```
                         ┌─────────────┐
                         │  Boas-vindas│
                         │ Login/Cadas.│
                         └──────┬──────┘
                                │
                         ┌──────▼──────┐
                         │    Início   │ (Dashboard)
                         └──────┬──────┘
      ┌───────────────┬─────────┼──────────────┬────────────────┐
┌─────▼─────┐   ┌─────▼─────┐ ┌─▼────────┐ ┌───▼───────┐  ┌─────▼─────┐
│ Guarda-   │   │Marketplace│ │ Looks    │ │ Meus      │  │  Perfil   │
│ roupa     │   │           │ │          │ │ anúncios  │  │           │
└─────┬─────┘   └─────┬─────┘ └─┬────────┘ └───┬───────┘  └───────────┘
 ┌────┼─────┐         │         │          ┌───┴────┐
 │Cad.│Det. │    Detalhe do   Criar/      Propostas  Novo
 │peça│peça │    anúncio      ver look    de troca   anúncio
 └────┴──┬──┘      │ │
         │         │ └──► Proposta de troca
    Criar anúncio  └────► Checkout simulado
```

**Barra de navegação inferior (fixa, 5 itens):** Início · Guarda-roupa · **＋** (ação rápida: cadastrar peça) · Marketplace · Perfil

---

## 6. Sumário das telas

| # | Tela | Rota | Casos de uso | Elementos principais |
|---|---|---|---|---|
| T01 | Boas-vindas / Login / Cadastro | `#/login` | UC01 | Logo, abas Entrar/Criar conta, campos, botão |
| T02 | Início (Dashboard) | `#/inicio` | UC10 | Saudação, resumo, alerta peças paradas, atalhos, destaques |
| T03 | Meu Guarda-roupa | `#/guarda-roupa` | UC03, UC04 | Busca, filtros (chips), grade de peças, contador, botão ＋ |
| T04 | Cadastrar / Editar peça | `#/peca/nova`, `#/peca/:id/editar` | UC02, UC05 | Upload de foto, formulário, seletores, botões salvar/cancelar |
| T05 | Detalhe da peça | `#/peca/:id` | UC05, UC06, UC09 | Foto, atributos, contador de usos, ações (usei, editar, anunciar, excluir) |
| T06 | Criar anúncio | `#/anuncio/novo?peca=:id` | UC06 | Peça selecionada, título, descrição, tipo, preço, simulador de ganho |
| T07 | Marketplace | `#/marketplace` | UC07 | Busca, filtros, ordenação, grade de anúncios, selo de destaque |
| T08 | Detalhe do anúncio | `#/anuncio/:id` | UC07, UC08 | Foto, preço, vendedor, selo, botões Comprar/Propor troca/Favoritar |
| T09 | Checkout simulado | `#/anuncio/:id/comprar` | UC08 | Resumo, endereço, forma de pagamento, total, confirmar |
| T10 | Proposta de troca | `#/anuncio/:id/trocar` | UC08 | Peça desejada, seleção de peça própria, mensagem, enviar |
| T11 | Meus anúncios | `#/meus-anuncios` | UC11 | Abas (Ativos/Pausados/Vendidos), cards, ações, propostas recebidas |
| T12 | Peças pouco usadas | `#/pecas-paradas` | UC09 | Lista de peças paradas, motivo, ações (anunciar/marcar uso) |
| T13 | Looks | `#/looks` | UC12 | Lista de looks, botão criar, sugestão automática |
| T14 | Montar look | `#/looks/novo` | UC12 | Seleção de peças por categoria, pré-visualização, salvar |
| T15 | Favoritos | `#/favoritos` | UC07 | Grade de anúncios favoritados |
| T16 | Perfil | `#/perfil` | UC13 | Dados, estatísticas, sair |

---

## 7. Casos de uso detalhados

### UC01 — Cadastrar-se e entrar (simulado)

| Campo | Descrição |
|---|---|
| **Ator** | Visitante |
| **Pré-condição** | Nenhuma |
| **Pós-condição** | Usuário logado; sessão salva em `localStorage`; redirecionado para T02 |
| **Telas** | T01 |

**Fluxo principal**
1. Visitante abre o app e vê a tela T01.
2. Seleciona a aba **Criar conta**.
3. Informa nome, e-mail, senha e cidade.
4. Clica em **Criar conta**.
5. Sistema valida os campos, salva o usuário e carrega os dados de exemplo (seed).
6. Sistema redireciona para **Início**.

**Fluxos alternativos**
- **A1 — Já possui conta:** na aba **Entrar**, informa e-mail e senha; sistema confere com o usuário salvo.
- **A2 — Entrar como demonstração:** botão "Explorar com conta demo" cria usuário fictício já com peças e anúncios.

**Fluxos de exceção**
- **E1:** campo vazio → mensagem de erro abaixo do campo.
- **E2:** e-mail inválido → "Informe um e-mail válido".
- **E3:** senha < 6 caracteres → "A senha deve ter ao menos 6 caracteres".
- **E4:** credenciais não conferem → "E-mail ou senha incorretos".

---

### UC02 — Cadastrar peça de roupa

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Pré-condição** | Logado |
| **Pós-condição** | Peça salva com status `no_armario` e visível no guarda-roupa |
| **Telas** | T03 → T04 → T03 |

**Fluxo principal**
1. Usuário toca em **＋** (barra inferior ou botão flutuante em T03).
2. Sistema abre T04 com o formulário em branco.
3. Usuário toca na área de foto e escolhe imagem (câmera/galeria).
4. Sistema exibe a pré-visualização (imagem redimensionada).
5. Usuário preenche: nome, categoria, cor, tamanho, estação(ões), marca (opcional), ocasião (opcional), estado.
6. Usuário toca em **Salvar peça**.
7. Sistema valida (RN01), gera `id`, define `usos = 0`, `dataCadastro = hoje`, `status = no_armario`.
8. Sistema exibe mensagem "Peça adicionada!" e retorna para T03 com a nova peça no topo.

**Fluxos alternativos**
- **A1 — Salvar e adicionar outra:** botão secundário limpa o formulário mantendo categoria e estação.
- **A2 — Sem foto:** permite salvar; usa ícone da categoria.

**Exceções**
- **E1:** campos obrigatórios vazios → destaca campos em vermelho e impede salvar.
- **E2:** imagem não é arquivo de imagem → "Arquivo inválido".
- **E3:** `localStorage` cheio → "Espaço insuficiente. Remova peças ou use fotos menores."

---

### UC03 — Visualizar guarda-roupa

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Pré-condição** | Logado |
| **Pós-condição** | Lista exibida |
| **Telas** | T03 |

**Fluxo principal**
1. Usuário acessa **Guarda-roupa** na barra inferior.
2. Sistema lista as peças com status `no_armario` ou `anunciada` em grade de 2 colunas.
3. Cada card mostra foto, nome, categoria/tamanho e etiqueta de status (ex.: "À venda").
4. Sistema exibe o total ("24 peças").
5. Usuário rola a lista; ao tocar em um card, abre T05 (Detalhe da peça).

**Alternativos**
- **A1 — Alternar visualização:** botões Grade / Lista.
- **A2 — Guarda-roupa vazio:** exibe ilustração + texto "Seu guarda-roupa está vazio" + botão **Adicionar primeira peça**.

---

### UC04 — Filtrar e buscar peças no guarda-roupa

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Pré-condição** | Existe ao menos uma peça |
| **Pós-condição** | Lista reduzida conforme filtros |
| **Telas** | T03 |

**Fluxo principal**
1. Usuário digita na barra de busca (filtra por nome/marca em tempo real).
2. Usuário toca em **Filtros**; abre painel (*bottom sheet*) com:
   - Categoria (múltipla escolha)
   - Estação
   - Cor
   - Tamanho
   - Ocasião
   - Status (No armário / À venda)
   - Uso (Todas / Pouco usadas / Mais usadas)
3. Usuário seleciona opções e toca em **Aplicar**.
4. Sistema atualiza a lista e mostra **chips** com os filtros ativos (cada um removível com ✕).
5. Contador mostra "8 de 24 peças".

**Alternativos**
- **A1 — Limpar filtros:** botão remove todos os chips.
- **A2 — Atalho de estação:** chips rápidos no topo (Verão, Inverno etc.).
- **A3 — Ordenar:** Mais recentes · Mais usadas · Menos usadas · A–Z.

**Exceção**
- **E1:** nenhum resultado → "Nenhuma peça encontrada" + botão **Limpar filtros**.

---

### UC05 — Ver, editar e excluir peça

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Pré-condição** | Peça existente do usuário |
| **Telas** | T05, T04 |

**Fluxo principal (ver)**
1. Usuário toca em uma peça no guarda-roupa.
2. Sistema abre T05 com foto grande, atributos, número de usos, último uso e status.

**Fluxo alternativo — Editar**
1. Em T05, toca em **Editar**.
2. Sistema abre T04 preenchida.
3. Usuário altera e toca **Salvar alterações**; sistema atualiza e volta para T05.

**Fluxo alternativo — Registrar uso**
1. Em T05, toca em **Usei hoje**.
2. Sistema incrementa `usos` e atualiza `ultimoUso = hoje`.

**Fluxo alternativo — Excluir**
1. Toca em **Excluir**; sistema abre confirmação "Excluir esta peça? Essa ação não pode ser desfeita".
2. Confirma → peça removida; retorna para T03.

**Exceção**
- **E1:** peça com anúncio ativo → bloqueia exclusão: "Pause ou exclua o anúncio antes de remover a peça".

---

### UC06 — Criar anúncio a partir de uma peça

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Pré-condição** | Peça com status `no_armario` (RN02, RN03) |
| **Pós-condição** | Anúncio `ativo` criado; peça passa a `anunciada` (RN04) |
| **Telas** | T05 / T12 → T06 → T11 |

**Fluxo principal**
1. Em T05 (ou em T12), usuário toca em **Anunciar esta peça**.
2. Sistema abre T06 com a foto e os dados da peça já preenchidos (título sugerido: "Categoria + Marca + Tamanho").
3. Usuário edita título e descrição.
4. Escolhe o **tipo**: Venda · Troca · Venda ou troca.
5. Se Venda/Ambos: informa o preço. Sistema exibe **simulador**:
   > Preço R$ 100,00 → comissão (10%) R$ 10,00 → **você recebe R$ 90,00**
6. Se Troca/Ambos: informa o que gostaria de receber (opcional).
7. Opcional: ativa **Destacar anúncio** (R$ 4,90 / 7 dias — simulado).
8. Toca em **Publicar anúncio**.
9. Sistema valida, cria o anúncio, atualiza o status da peça e mostra "Anúncio publicado!".
10. Redireciona para T11 (aba Ativos).

**Alternativos**
- **A1 — Salvar como rascunho:** (opcional no MVP) anúncio fica `pausado`.
- **A2 — Anunciar direto do marketplace:** botão "Anunciar" abre seletor de peças disponíveis.

**Exceções**
- **E1:** preço vazio ou < R$ 5,00 (RN12) → erro no campo.
- **E2:** peça já anunciada → botão exibe "Ver anúncio" em vez de "Anunciar".
- **E3:** título vazio → erro.

---

### UC07 — Visualizar e filtrar anúncios de outras pessoas

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Pré-condição** | Logado |
| **Pós-condição** | Anúncios exibidos |
| **Telas** | T07, T08, T15 |

**Fluxo principal**
1. Usuário acessa **Marketplace**.
2. Sistema lista os anúncios `ativos` **de outros usuários**, com os **destacados primeiro**.
3. Cada card exibe: foto, título, preço (ou "Troca"), tamanho, cidade e etiqueta (Destaque).
4. Usuário usa busca e filtros: categoria, tamanho, cor, faixa de preço, tipo (Venda/Troca), estado.
5. Ordena por: Mais recentes · Menor preço · Maior preço.
6. Toca em um card → T08 (detalhe).

**Alternativos**
- **A1 — Favoritar:** ícone de coração no card adiciona/remove de `favoritos`; lista em T15.
- **A2 — Ver perfil do vendedor:** em T08, toca no nome do vendedor e vê os demais anúncios dele.

**Exceção**
- **E1:** sem resultados → "Nenhum anúncio encontrado. Tente ampliar os filtros".

---

### UC08 — Comprar ou propor troca (simulado)

| Campo | Descrição |
|---|---|
| **Ator** | Usuário (comprador) |
| **Pré-condição** | Anúncio ativo de outro usuário (RN09) |
| **Telas** | T08 → T09 (compra) ou T10 (troca) |

**Fluxo principal — Compra**
1. Em T08, toca em **Comprar**.
2. Sistema abre T09 com resumo: item, preço, taxa de envio simulada (valor fixo, ex.: R$ 15,00), total.
3. Usuário preenche endereço e escolhe pagamento (Pix / Cartão — apenas visual).
4. Toca em **Confirmar compra**.
5. Sistema registra a `transação`, calcula a comissão (RN05/RN06), muda o anúncio para `vendido` e a peça para `vendida` (RN11).
6. Exibe tela de sucesso com número do pedido e botão **Voltar ao marketplace**.

**Fluxo alternativo — Proposta de troca**
1. Em T08, toca em **Propor troca** (disponível para tipo `troca` ou `ambos`).
2. Sistema abre T10 com a peça desejada no topo e a lista das **peças próprias** disponíveis.
3. Usuário seleciona uma peça e (opcional) escreve uma mensagem.
4. Toca em **Enviar proposta**; sistema cria a proposta com status `pendente`.
5. O vendedor vê a proposta em T11 e pode **Aceitar** ou **Recusar** (RN13).

**Exceções**
- **E1:** endereço incompleto → impede confirmar.
- **E2:** usuário sem peças disponíveis → "Você não tem peças disponíveis para trocar" + link para cadastrar.
- **E3:** anúncio vendido por outra pessoa enquanto navegava → "Este anúncio não está mais disponível".

---

### UC09 — Ver peças pouco utilizadas (proposta de valor)

| Campo | Descrição |
|---|---|
| **Ator** | Usuário / Sistema |
| **Pré-condição** | Peças cadastradas |
| **Telas** | T02 (alerta) → T12 |

**Fluxo principal**
1. Ao abrir o **Início**, sistema avalia as peças conforme RN08.
2. Se houver peças paradas, exibe um card: "Você tem **6 peças** pouco usadas".
3. Usuário toca no card e abre T12.
4. Sistema lista as peças com o motivo ("Usada 1x em 8 meses" / "Sem uso há 210 dias").
5. Para cada peça, o usuário pode: **Anunciar**, **Registrar uso** ou **Manter** (oculta o aviso por 30 dias).

**Alternativo**
- **A1:** nenhuma peça parada → "Parabéns! Você aproveita bem seu guarda-roupa 🎉".

---

### UC10 — Visualizar o Início (dashboard)

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Telas** | T02 |

**Fluxo principal**
1. Após login, sistema exibe saudação com o nome.
2. Mostra os indicadores: total de peças, peças à venda, peças pouco usadas, vendas realizadas.
3. Exibe o alerta de UC09 (se aplicável).
4. Exibe atalhos rápidos: **Adicionar peça**, **Montar look**, **Ver marketplace**.
5. Exibe carrossel "Destaques do marketplace" (3–5 anúncios destacados).

---

### UC11 — Gerenciar meus anúncios

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Telas** | T11 |

**Fluxo principal**
1. Usuário acessa **Meus anúncios** (via Perfil ou Início).
2. Sistema mostra abas: **Ativos · Pausados · Vendidos/Trocados · Propostas**.
3. Cada card exibe foto, título, preço, tipo, visualizações (contador simples) e favoritos.
4. Ações por anúncio: **Editar**, **Pausar/Reativar**, **Destacar**, **Excluir**.

**Alternativos**
- **A1 — Propostas de troca:** na aba Propostas, usuário vê a peça oferecida e escolhe **Aceitar** ou **Recusar**.
- **A2 — Excluir:** pede confirmação; peça volta a `no_armario` (RN04).

---

### UC12 — Montar looks (sugestão de looks)

| Campo | Descrição |
|---|---|
| **Ator** | Usuário / Sistema |
| **Telas** | T13, T14 |

**Fluxo principal — manual**
1. Usuário acessa **Looks** e toca em **Criar look**.
2. Em T14, escolhe peças por categoria (Parte de cima, Parte de baixo, Calçado, Acessório).
3. A pré-visualização monta uma "colagem" simples com as fotos.
4. Informa o nome do look e toca em **Salvar look**.

**Fluxo alternativo — sugestão automática (regra simples, sem IA)**
1. Usuário toca em **Sugerir look**.
2. Sistema escolhe uma peça de cima + uma de baixo + um calçado, **priorizando peças pouco usadas** e da **estação atual**.
3. Usuário pode **Aceitar**, **Trocar peça** ou **Gerar outro**.
4. Ao aceitar o look e tocar **Usei este look**, sistema incrementa `usos` de todas as peças.

---

### UC13 — Gerenciar perfil

| Campo | Descrição |
|---|---|
| **Ator** | Usuário |
| **Telas** | T16 |

**Fluxo principal**
1. Usuário acessa **Perfil**.
2. Sistema exibe os dados do usuário.
3. Usuário edita nome, cidade e foto.
4. Toca em **Sair** → limpa sessão e volta para T01.

**Alternativo**
- **A1 — Resetar dados da demo:** botão que limpa `localStorage` e recarrega o seed.

---

## 8. Layouts das telas (wireframes)

> Wireframes em baixa fidelidade, largura de referência 390 px. `[ ]` = campo/botão · `( )` = opção · `▢` = imagem.

### T01 — Boas-vindas / Login / Cadastro

```
┌──────────────────────────────┐
│                              │
│           👗 LOGO            │
│      Nome do App             │
│  Seu guarda-roupa, sem       │
│  roupa parada.               │
│                              │
│  [  Entrar  |  Criar conta ] │
│  ─────────────────────────── │
│  Nome                        │
│  [___________________]       │
│  E-mail                      │
│  [___________________]       │
│  Senha                       │
│  [___________________] 👁    │
│  Cidade                      │
│  [___________________]       │
│                              │
│  [      CRIAR CONTA      ]   │
│  [ Explorar com conta demo ] │
└──────────────────────────────┘
```

**Elementos:** logo, slogan, abas, campos (nome, e-mail, senha, cidade), botão primário, botão demo, mensagens de erro.

---

### T02 — Início

```
┌──────────────────────────────┐
│ Olá, Maria 👋        🔔      │
├──────────────────────────────┤
│ ┌──────┐┌──────┐┌──────┐┌────┐│
│ │  24  ││  3   ││  6   ││ 2  ││
│ │peças ││à venda││paradas││vend.││
│ └──────┘└──────┘└──────┘└────┘│
│                              │
│ ┌──────────────────────────┐ │
│ │ ⚠ 6 peças pouco usadas   │ │
│ │ Que tal vender ou trocar?│ │
│ │            [ Ver peças ] │ │
│ └──────────────────────────┘ │
│                              │
│ Atalhos                      │
│ [＋ Peça] [👔 Look] [🛍 Loja]│
│                              │
│ Destaques do marketplace  >  │
│ ┌────┐ ┌────┐ ┌────┐        │
│ │ ▢  │ │ ▢  │ │ ▢  │ →      │
│ │R$80│ │Troca│ │R$45│        │
│ └────┘ └────┘ └────┘        │
├──────────────────────────────┤
│ 🏠   👚   ＋   🛍   👤        │
└──────────────────────────────┘
```

**Elementos:** saudação, ícone de notificações (propostas), 4 indicadores, card de alerta (condicional), atalhos, carrossel de destaques, barra de navegação.

---

### T03 — Meu Guarda-roupa

```
┌──────────────────────────────┐
│ Meu guarda-roupa      ▦ ☰   │
├──────────────────────────────┤
│ 🔍 [Buscar peça...        ]  │
│ [⚙ Filtros (2)] [Ordenar ▾]  │
│ (Verão✕) (Camisetas✕)        │
│ 8 de 24 peças                │
│                              │
│ ┌────────────┐ ┌────────────┐│
│ │            │ │            ││
│ │     ▢      │ │     ▢      ││
│ │            │ │  [À venda] ││
│ │ Camiseta   │ │ Short jeans││
│ │ branca · M │ │ Azul · 38  ││
│ └────────────┘ └────────────┘│
│ ┌────────────┐ ┌────────────┐│
│ │     ▢      │ │     ▢      ││
│ │ ...        │ │ ...        ││
│ └────────────┘ └────────────┘│
│                          (＋)│
├──────────────────────────────┤
│ 🏠   👚   ＋   🛍   👤        │
└──────────────────────────────┘
```

**Elementos:** título, alternador grade/lista, busca, botão Filtros com contador, ordenação, chips ativos removíveis, contador, cards (foto, nome, cor, tamanho, etiqueta de status), botão flutuante ＋, estado vazio.

**Painel de filtros (bottom sheet):**

```
┌──────────────────────────────┐
│ ▬▬▬                          │
│ Filtros              Limpar  │
│ Categoria                    │
│ (Camisetas)(Calças)(Vestidos)│
│ Estação                      │
│ (Primavera)(Verão)(Outono)...│
│ Cor                          │
│ ●  ●  ●  ●  ●  ●  ●          │
│ Tamanho                      │
│ (P)(M)(G)(GG)                │
│ Uso                          │
│ (Todas)(Pouco usadas)(Mais…) │
│                              │
│ [     APLICAR (8 peças)    ] │
└──────────────────────────────┘
```

---

### T04 — Cadastrar / Editar peça

```
┌──────────────────────────────┐
│ ←  Nova peça                 │
├──────────────────────────────┤
│ ┌──────────────────────────┐ │
│ │        📷                │ │
│ │  Toque para adicionar    │ │
│ │       uma foto           │ │
│ └──────────────────────────┘ │
│ Nome da peça *               │
│ [________________________]   │
│ Categoria *                  │
│ [Selecione            ▾ ]    │
│ Cor *            Tamanho *   │
│ [Selecione ▾]    [ M ▾ ]     │
│ Estação * (múltipla escolha) │
│ [Primavera][Verão][Outono]   │
│ [Inverno][Todas]             │
│ Marca (opcional)             │
│ [________________________]   │
│ Ocasião                      │
│ [Casual ▾]                   │
│ Estado                       │
│ (•)Novo ( )Seminovo ( )Bom   │
│                              │
│ [     SALVAR PEÇA          ] │
│ [  Salvar e adicionar outra] │
└──────────────────────────────┘
```

**Elementos:** header com voltar, área de upload com pré-visualização, campos obrigatórios (*), seletor múltiplo de estação, botões de ação, mensagens de validação.

---

### T05 — Detalhe da peça

```
┌──────────────────────────────┐
│ ←  Detalhes            ✏  🗑 │
├──────────────────────────────┤
│ ┌──────────────────────────┐ │
│ │                          │ │
│ │           ▢              │ │
│ │                          │ │
│ └──────────────────────────┘ │
│ Jaqueta jeans   [No armário] │
│ Levi's · Casual              │
│                              │
│ Categoria     Casacos        │
│ Cor           Azul           │
│ Tamanho       M              │
│ Estação       Outono, Inverno│
│ Estado        Seminovo       │
│ ──────────────────────────── │
│ Usada 2x · último uso 10/03  │
│ ⚠ Pouco utilizada            │
│                              │
│ [      ✔ USEI HOJE        ]  │
│ [   💰 ANUNCIAR ESTA PEÇA ]  │
└──────────────────────────────┘
```

**Elementos:** foto grande, nome, etiqueta de status, atributos, estatísticas de uso, aviso "pouco utilizada" (condicional), botões Usei hoje / Anunciar (ou "Ver anúncio"), ícones de editar e excluir.

---

### T06 — Criar anúncio

```
┌──────────────────────────────┐
│ ←  Novo anúncio              │
├──────────────────────────────┤
│ ┌────┐ Jaqueta jeans         │
│ │ ▢  │ Casacos · M · Azul    │
│ └────┘                       │
│ Título *                     │
│ [Jaqueta jeans Levi's M   ]  │
│ Descrição                    │
│ [________________________]   │
│ [________________________]   │
│ Tipo de anúncio              │
│ (•) Venda ( ) Troca ( ) Ambos│
│ Preço (R$) *                 │
│ [ 120,00 ]                   │
│ ┌──────────────────────────┐ │
│ │ Comissão (10%) −R$ 12,00 │ │
│ │ Você recebe  R$ 108,00   │ │
│ └──────────────────────────┘ │
│ O que aceita em troca?       │
│ [________________________]   │
│ [✓] Destacar (R$ 4,90/7 dias)│
│                              │
│ [    PUBLICAR ANÚNCIO     ]  │
└──────────────────────────────┘
```

**Elementos:** resumo da peça, título, descrição, seletor de tipo, preço (condicional), simulador de comissão, campo de preferência de troca (condicional), opção de destaque, botão publicar.

---

### T07 — Marketplace

```
┌──────────────────────────────┐
│ Marketplace            ♡     │
├──────────────────────────────┤
│ 🔍 [Buscar roupas...       ]  │
│ [⚙ Filtros] [Ordenar ▾]      │
│ (Todos)(Vestidos)(Casacos)...│
│                              │
│ ┌────────────┐ ┌────────────┐│
│ │ ★Destaque ♡│ │          ♡ ││
│ │     ▢      │ │     ▢      ││
│ │ Jaqueta M  │ │ Vestido P  ││
│ │ R$ 120     │ │ 🔄 Troca   ││
│ │ Passo Fundo │ │ Carazinho   ││
│ └────────────┘ └────────────┘│
│ ┌────────────┐ ┌────────────┐│
│ │     ▢      │ │     ▢      ││
│ │ ...        │ │ ...        ││
│ └────────────┘ └────────────┘│
├──────────────────────────────┤
│ 🏠   👚   ＋   🛍   👤        │
└──────────────────────────────┘
```

**Elementos:** busca, filtros (categoria, tamanho, cor, preço, tipo, estado), ordenação, chips de categoria rápida, cards com foto/título/preço ou "Troca"/tamanho/cidade, etiqueta "Destaque", ícone de favorito, acesso aos favoritos.

---

### T08 — Detalhe do anúncio

```
┌──────────────────────────────┐
│ ←                  ♡   ⤴    │
├──────────────────────────────┤
│ ┌──────────────────────────┐ │
│ │            ▢             │ │
│ └──────────────────────────┘ │
│ Jaqueta jeans Levi's M       │
│ R$ 120,00     [★ Destaque]   │
│ Também aceita troca          │
│ ──────────────────────────── │
│ Tamanho M · Azul · Seminovo  │
│ Categoria Casacos            │
│ Descrição:                   │
│ Pouco usada, sem manchas...  │
│ ──────────────────────────── │
│ 👤 Brechó da Ana             │
│ 📍 Passo Fundo · 12 anúncios │
│                              │
│ [        COMPRAR         ]   │
│ [      PROPOR TROCA      ]   │
└──────────────────────────────┘
```

**Elementos:** foto, título, preço, etiquetas (destaque/tipo), atributos, descrição, card do vendedor (nome, selo, cidade, nº de anúncios), botões Comprar e Propor troca (condicionais ao tipo), favoritar, compartilhar (copiar link).

---

### T09 — Checkout simulado

```
┌──────────────────────────────┐
│ ←  Finalizar compra          │
├──────────────────────────────┤
│ ┌────┐ Jaqueta jeans Levi's  │
│ │ ▢  │ R$ 120,00             │
│ └────┘                       │
│ Endereço de entrega          │
│ [Rua e número            ]   │
│ [Cidade       ][CEP       ]  │
│ Pagamento                    │
│ (•) Pix  ( ) Cartão          │
│ ──────────────────────────── │
│ Produto            R$ 120,00 │
│ Frete (simulado)   R$  15,00 │
│ Total              R$ 135,00 │
│                              │
│ [     CONFIRMAR COMPRA    ]  │
└──────────────────────────────┘
```

**Tela de sucesso:** ✔ ícone, "Compra realizada!", nº do pedido, botão "Voltar ao marketplace".

---

### T10 — Proposta de troca

```
┌──────────────────────────────┐
│ ←  Propor troca              │
├──────────────────────────────┤
│ Você quer:                   │
│ ┌────┐ Vestido floral P      │
│ │ ▢  │ de Carla              │
│ └────┘                       │
│ Escolha uma peça sua:        │
│ ┌──────┐┌──────┐┌──────┐     │
│ │ ▢ ◉  ││  ▢   ││  ▢   │     │
│ │Blusa ││Saia  ││Short │     │
│ └──────┘└──────┘└──────┘     │
│ Mensagem (opcional)          │
│ [________________________]   │
│                              │
│ [     ENVIAR PROPOSTA     ]  │
└──────────────────────────────┘
```

---

### T11 — Meus anúncios

```
┌──────────────────────────────┐
│ ←  Meus anúncios             │
├──────────────────────────────┤
│ [Ativos][Pausados][Vendidos] │
│ [Propostas (2)]              │
│                              │
│ ┌──────────────────────────┐ │
│ │ ┌────┐ Jaqueta jeans     │ │
│ │ │ ▢  │ R$ 120 · Venda    │ │
│ │ └────┘ 👁 34  ♡ 5        │ │
│ │ [Editar][Pausar][Destacar]│ │
│ └──────────────────────────┘ │
│ ┌──────────────────────────┐ │
│ │ ...                      │ │
│ └──────────────────────────┘ │
└──────────────────────────────┘
```

**Aba Propostas:** card com peça desejada ↔ peça oferecida, mensagem, botões **Aceitar** / **Recusar**.

---

### T12 — Peças pouco usadas

```
┌──────────────────────────────┐
│ ←  Peças paradas             │
├──────────────────────────────┤
│ 6 peças pouco utilizadas     │
│ ┌──────────────────────────┐ │
│ │ ┌────┐ Casaco de lã      │ │
│ │ │ ▢  │ Usado 1x em 8 meses│ │
│ │ └────┘                   │ │
│ │ [Anunciar][Usei][Manter] │ │
│ └──────────────────────────┘ │
│ ┌──────────────────────────┐ │
│ │ ...                      │ │
│ └──────────────────────────┘ │
└──────────────────────────────┘
```

---

### T13 — Looks

```
┌──────────────────────────────┐
│ Looks                        │
├──────────────────────────────┤
│ [ ✨ Sugerir look ]          │
│ [ ＋ Criar look ]            │
│ Meus looks                   │
│ ┌──────────┐ ┌──────────┐    │
│ │ ▢ ▢ ▢    │ │ ▢ ▢ ▢    │    │
│ │ Trabalho │ │ Fim de   │    │
│ │          │ │ semana   │    │
│ └──────────┘ └──────────┘    │
└──────────────────────────────┘
```

### T14 — Montar look

```
┌──────────────────────────────┐
│ ←  Novo look                 │
├──────────────────────────────┤
│ ┌──────────────────────────┐ │
│ │   Pré-visualização       │ │
│ │     ▢   ▢   ▢           │ │
│ └──────────────────────────┘ │
│ Parte de cima    →→ ▢ ▢ ▢    │
│ Parte de baixo   →→ ▢ ▢ ▢    │
│ Calçado          →→ ▢ ▢ ▢    │
│ Acessório        →→ ▢ ▢ ▢    │
│ Nome do look                 │
│ [________________________]   │
│ [        SALVAR LOOK      ]  │
└──────────────────────────────┘
```

---

### T15 — Favoritos

Grade de 2 colunas idêntica à T07, contendo apenas anúncios favoritados; coração preenchido remove o item. Estado vazio: "Você ainda não favoritou nenhum anúncio".

### T16 — Perfil

```
┌──────────────────────────────┐
│ Perfil                       │
├──────────────────────────────┤
│        ( 👤 )                │
│        Maria Silva           │
│     Passo Fundo - RS         │
│ ┌────────┐┌────────┐┌──────┐ │
│ │24 peças││3 anúnc.││2 vend.│ │
│ └────────┘└────────┘└──────┘ │
│ > Meus anúncios              │
│ > Favoritos                  │
│ > Peças pouco usadas         │
│ > Editar perfil              │
│ > Resetar dados da demo      │
│ [          SAIR           ]  │
└──────────────────────────────┘
```

---

## 9. Componentes reutilizáveis

| Componente | Uso |
|---|---|
| **Header** (voltar + título + ações) | Todas as telas internas |
| **Barra de navegação inferior** | T02, T03, T07, T16 |
| **Card de peça** | T03, T12, T14 |
| **Card de anúncio** | T07, T11, T15 |
| **Chip** (filtro/categoria/estação) | T03, T04, T07 |
| **Bottom sheet** | Filtros |
| **Modal de confirmação** | Excluir peça/anúncio |
| **Toast** (mensagem rápida) | Sucesso/erro após ações |
| **Estado vazio** | Listas sem itens |
| **Etiquetas de status** | "À venda", "Destaque", "Pouco usada" |

---

## 10. Requisitos não funcionais do MVP

- Funcionar em navegadores modernos (Chrome, Edge, Firefox, Safari) sem servidor (abrir `index.html`).
- Carregar em menos de 2 s com dados de exemplo.
- Acessibilidade básica: `alt` nas imagens, contraste adequado, botões com mínimo de 44 px de altura, `label` associado a cada campo.
- Mensagens de erro claras e em português.
- Dados persistidos entre recarregamentos da página.

---

## 11. Critérios de aceite (resumo)

| Caso de uso | Critério |
|---|---|
| UC02 | Peça cadastrada aparece no guarda-roupa após salvar e persiste após recarregar a página. |
| UC04 | Combinar ≥ 2 filtros reduz a lista corretamente e o contador reflete o resultado. |
| UC06 | Peça anunciada exibe a etiqueta "À venda" e não pode ser anunciada novamente. |
| UC06 | Simulador de comissão mostra 10% sobre o preço de venda. |
| UC07 | Marketplace nunca exibe anúncios do próprio usuário. |
| UC08 | Após a compra, anúncio e peça mudam para `vendido`/`vendida`. |
| UC09 | Peças que atendem à RN08 aparecem em T12 com o motivo correto. |
| UC11 | Excluir um anúncio devolve a peça ao status `no_armario`. |

---

## 12. Sugestão de ordem de implementação

1. **Base:** estrutura HTML, roteamento por hash, módulo de `localStorage`, seed de dados, barra de navegação.
2. **Sprint 1:** T01 → T03 → T04 → T05 (cadastro e guarda-roupa).
3. **Sprint 2:** filtros e busca (UC04) + T02 (início) + T12 (peças paradas).
4. **Sprint 3:** T06 → T11 (criar e gerenciar anúncios).
5. **Sprint 4:** T07 → T08 → T09 → T10 (marketplace, compra e troca simuladas).
6. **Sprint 5:** T13 → T14 (looks), T15, T16 e ajustes de UX.

---

## 13. Pontos em aberto

- Nome e identidade visual do app (paleta, tipografia, logo).
- A comissão de 10% e o preço do destaque (R$ 4,90) são valores sugeridos; ajuste conforme sua análise financeira.
- O item **"Venda de dados"** do Canvas foi **deixado fora do MVP** por questões éticas/LGPD; se mantido no plano de negócio, considerar apenas dados agregados e anonimizados, com consentimento.
