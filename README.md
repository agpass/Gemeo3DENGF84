# Gêmeo 3D — Simulador de Filas em 3D (Supermercado)

ENGF84 · Modelagem e Otimização de Sistemas de Produção · UFBA 2026.1 · Grupo 6 · Etapa 3

Visualização 3D do modelo de filas do supermercado, em **projeto separado** do simulador
([agpass/SupermercadoENGF84](https://github.com/agpass/SupermercadoENGF84)). Não altera o
original: lê o motor publicado, roda a simulação e mostra o resultado.

## O que a página faz

Ao abrir, aparece uma **tela de upload**. Você carrega uma planilha compatível com o simulador
(`.csv` ou `.xlsx`) — ou usa o botão de dados de exemplo. Então o gêmeo:

1. **Puxa o motor** do repositório do simulador (commit atual de `main`, via raw do GitHub) e o
   carrega em memória — nenhuma linha do motor está copiada aqui;
2. **Roda o pipeline do original** sobre a sua planilha: `parseData → limpar → estimar →
   prepararReais → calibraPacienciaReal → runCenario(dia real, seed 42)`;
3. Mostra **apenas duas coisas**: a **simulação em 3D** (com controle para acelerar o tempo) e um
   **relatório** com os parâmetros que o modelo encontrou nos seus dados.

Qualquer commit novo no simulador é refletido automaticamente no próximo carregamento.

## Planilha compatível (contrato de esquema)

```
id, dia, itens, chegada, coleta_inicio, coleta_fim, caixa_inicio, caixa_fim, abandonou
```

Datas em `AAAA-MM-DD HH:MM:SS`; `abandonou` em `True`/`False`. Qualquer log com esse esquema é
aceito e recalibra o modelo sozinho. Use "Trocar planilha" para carregar outra a qualquer momento.

## O relatório de parâmetros

O painel lateral lista o que o modelo estimou dos seus dados: registros limpos, dias, janela de
operação, caixas observados (c), μ por caixa, regressão do serviço (b0 + b1·itens), coleta média,
cesta média, λ de pico, abandono observado e paciência calibrada. Abaixo, os KPIs do dia
selecionado calculados pelo motor (ρ, Lq, Wq, abandono) e a validação abandono-modelo × real.

## Controle de tempo

Barra inferior: **play/pausa**, reiniciar e velocidade do relógio simulado — de **1 min/s** até
**2 h/s**. Também dá para trocar o **dia** (cada dia da planilha) e o **nº de caixas** (cenário
what-if re-executado pelo motor).

## O ambiente 3D

Mercado de porte médio, com **grade de corredores e prateleiras**: 6 gôndolas duplas em dois
segmentos (com corredor no meio), hortifrúti na entrada, parede de frios ao fundo, padaria, fila
única serpenteada e bateria de caixas com esteira e lâmpada livre/ocupado.

Movimentação e não-colisão:

- **Grade (roteamento Manhattan).** Todo deslocamento segue os corredores em segmentos ortogonais
  (nunca na diagonal), então **nenhuma entidade atravessa prateleiras**.
- **Fila única por slots.** Na fila FIFO cada cliente ocupa um slot fixo da serpentina, então não há
  sobreposição; a frente fica junto aos caixas e a cauda junto à entrada.
- **Separação anti-colisão.** Fora da fila, uma resolução por-quadro afasta clientes que fiquem
  próximos, sempre ao longo do eixo do corredor — evita que os corpos se sobreponham sem empurrá-los
  para dentro das prateleiras.

O trajeto de cada cliente vem dos **dados dele**: entra na `tChegada` real, faz um número de paradas
nas gôndolas proporcional aos `itens` da cesta (ocupando a janela real de coleta), entra na fila, é
atendido no caixa designado pelo modelo e sai — ou, se abandona, **larga o carrinho** e vai embora.

Cores: azul = coletando · âmbar = na fila · verde = em atendimento · vermelho = abandono.

Navegação: **órbita** (arrastar gira, roda dá zoom, botão direito move o alvo) e **modo andar** em
primeira pessoa (WASD/setas + mouse; Esc sai).

## Publicar

1. Crie um repositório novo, **público** (ex.: `gemeo3d-supermercado`);
2. Suba `index.html` e `README.md` na raiz;
3. **Settings → Pages → Deploy from a branch → main / (root)**;
4. Em ~1 min: `https://SEU-USUARIO.github.io/gemeo3d-supermercado/`.

Precisa de internet para puxar o motor do repositório do simulador. Se o CSV do simulador estiver
inacessível ao usar "dados de exemplo", o gerador do próprio motor é usado como fallback.
