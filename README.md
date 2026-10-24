# As Aventuras do Davizinho — Davi e sua Turma (SEGA Mega Drive / Genesis)

Jogo de plataforma 2D em C com o **SGDK**, tela 320×224, pixel art 16 bits, em português do Brasil.
Todos os gráficos, a música (PSG) e os efeitos sonoros são gerados por `tools/gerar_assets.py`.

> **Status (v2.0 - versao final):** 10 fases, 3 chefes, 6 tipos de inimigos, 16 musicas e todos os menus.
> A ROM **nao e gerada nem incluida neste ZIP** (o autor nao tem o SGDK): compile com `build.bat`.
> As versoes 1.0/1.1 foram compiladas e executadas pelo usuario (capturas de emulador). A logica da v2.0 foi
> **testada no PC** (ver "Testes"), mas **nao foi executada em emulador, nem compilada com o SGDK real**.

## Fases (10)
| # | Fase | Destaque |
|---|---|---|
| 1 | Bairro das Mangueiras | introducao |
| 2 | Praca Central | espinhos e bancos |
| 3 | Grande Mangueira | fase vertical (galhos) |
| 4 | Parquinho da Turma | **Chefe: Caramujao Rei** |
| 5 | Quintal do Vovo | plantas carnivoras, sapos, caixotes |
| 6 | Feira Livre | ratos, caixotes, espinhos |
| 7 | Rio das Pedras | rios e **Chefe: Abelhona Rainha** |
| 8 | Montanha dos Ventos | segunda fase vertical (nuvens) |
| 9 | Floresta Escura | morcegos, sapos, plantas (noite) |
| 10 | Aventura Final | **Chefe final: Rei Espinho** |

Fases 1-4 e 9 sao desenhadas a mao em `src/levels.c`; as fases 5, 6, 7, 8 e 10 sao geradas por regras em
`tools/gerar_fases.py` (-> `src/levels_gen.c`), que garantem vaos de 2 celulas, plataformas a 2 celulas de altura e
inimigos longe de buracos. Rode o script de novo para regenerar.

## Inimigos e chefes
- **Inimigos:** caramujo (anda), abelha (voa em onda), rato (rapido), morcego (mergulha), planta carnivora (so
  machuca aberta; pisar nela machuca), sapo (pula em direcao ao Davi). Pise ou use a estrela (A).
- **Chefes** (barra de vida no HUD, arena = ultima tela da fase, camera presa, a bandeira so aparece depois):
  - *Caramujao Rei* (3 vidas): vulneravel andando; ao ser atingido entra no casco girando (invulneravel).
  - *Abelhona Rainha* (4 vidas): paira, mergulha sobre o Davi e fica atordoada no chao (so ai e vulneravel).
  - *Rei Espinho* (6 vidas): persegue e pula; fica de espinhos (invulneravel); abaixo de 3 vidas lanca espinhos no chao.
  Se o Davi perde uma vida na luta, ela recomeca com o chefe cheio.
- **Musica:** uma faixa por fase (compostas por `tools/gerar_assets.py`), tema do titulo, dois temas de chefe,
  creditos, vitoria e game over (PSG). Sprites do Davi, inimigos, chefes e tiles tambem sao gerados pelo script.

## Menus
| Item do titulo | O que faz |
|---|---|
| NOVO JOGO | Comeca na Fase 1 |
| CONTINUAR (SENHA) | Digite a senha de uma fase para continuar dela (CIMA/BAIXO = letra, ESQ/DIR = posicao, START = OK, B = voltar) |
| SELECIONAR FASE | Escolhe entre as fases ja liberadas (no inicio so a Fase 1) |
| OPCOES | Musica LIGADA/DESLIGADA, Efeitos LIGADOS/DESLIGADOS, Dificuldade, Como jogar (controles) |
| CREDITOS | Mostra os creditos |

**Dificuldade:** FACIL = 5 vidas e 4 coracoes; NORMAL = 3 vidas e 3 coracoes; DIFICIL = 2 vidas e 2 coracoes.

**Senhas** (aparecem na tela "FASE CONCLUIDA!" ao terminar cada fase): Fase 1 `DAVI`, 2 `MANGA`, 3 `PRACA`, 4 `ARVORE`,
5 `PARQUE`, 6 `QUINTAL`, 7 `FEIRA`, 8 `RIO`, 9 `MONTANHA`, 10 `FINAL`. Opcoes e fases liberadas ficam so na memoria (nao ha save em cartucho); a senha serve
para retomar depois de desligar. **Pausa:** START continua, A reinicia a fase, B volta ao menu.

## Fluxo da campanha
```
Titulo -> Novo Jogo -> [Cartao da fase] -> Fase 1 -> "FASE CONCLUIDA!" -> Fase 2 -> ... -> Fase 10 (chefe final)
       -> Cena final -> "PARABENS! VOCE COMPLETOU AS AVENTURAS DO DAVIZINHO!" -> Creditos
       -> Opcoes: VOLTAR AO TITULO / JOGAR NOVAMENTE
```
- Cada fase tem comprimento limitado, camera que para nas bordas do mapa, paredes laterais e uma **bandeira** no
  fim. Moedas, tempo e rolagem **nao** concluem a fase: so tocar a bandeira (`SP_GOAL` -> `player_win`).
- **Selecionar Fase:** no inicio so a Fase 1 esta liberada; cada fase concluida libera a seguinte (progresso mantido
  enquanto o console estiver ligado).
- **Derrota:** o Davi tem 3 de HP (coracoes); dano tira 1 HP (com invulnerabilidade temporaria), espinhos tambem
  machucam; HP 0 ou cair em buraco = perde 1 vida e renasce no ultimo checkpoint (ou no inicio da fase). Sem vidas:
  **GAME OVER** com **TENTAR NOVAMENTE** (mesma fase, 3 vidas) ou **VOLTAR AO MENU**.
- O personagem nao sai do mapa (paredes laterais + limite de x) e cair abaixo do mapa tira uma vida.
- Textos sem acentos (a fonte padrao do SGDK so tem ASCII): "FASE CONCLUIDA!", "PARABENS!", etc.

## Testes automatizados (PC)
`sh tests/run_tests.sh` (requer gcc). Compila o codigo real de `src/` contra um console simulado (`tests/stub`) e roda
um bot que joga usando a propria fisica do jogo. Cenarios: campanha completa 1->10->final->creditos->titulo (inclui os 3 chefes);
Game Over -> menu; Game Over -> tentar novamente; jogar novamente apos o final; selecao so com a Fase 1 liberada
(com toques segurados por 8 quadros, como uma pessoa); senha valida e invalida; opcoes (musica, efeitos,
dificuldade); pausa -> menu; creditos pelo titulo; morte e respawn no checkpoint; luta contra cada chefe (fases 4, 7 e 10) e a verificacao de que a bandeira
nunca vale com o chefe vivo. Tambem verifica a cada quadro que camera e jogador ficam dentro do mapa.
Limites: nao testa graficos, som, tempo real nem a API real do SGDK, e o bot nao substitui jogadores humanos.

## Requisitos (Windows)
1. **SGDK 2.x** — https://github.com/Stephane-D/SGDK (use a release para Windows; inclui `make.exe`, o compilador m68k e o rescomp; Java é exigido pelo SGDK para algumas ferramentas, conforme a documentação dele).
2. **Python 3 + Pillow** (`pip install pillow`) — só para regenerar os assets.
3. Um emulador: BlastEm, Gens, Kega Fusion ou similar.

### Variável de ambiente GDK
Aponte `GDK` para a pasta do SGDK (a que contém `bin\make.exe` e `makefile.gen`):

```
setx GDK C:\sgdk
```
Feche e reabra o terminal. Confira com `echo %GDK%`.

## Compilar
Na pasta do projeto:
```
python tools\gerar_assets.py     (opcional: o build.bat gera se os PNGs não existirem)
build.bat
```
A ROM é gerada em **`out\rom.bin`** (convenção do `makefile.gen` do SGDK). `build.bat clean` limpa.
Abra `out\rom.bin` no emulador.

## Controles (controle de 3 botões)
| Botão | Ação |
|---|---|
| Direcional | Andar (acelera até correr) |
| B ou C | Pular (segure para pular mais alto) |
| A | Lançar estrela (derrota inimigos à distância) |
| START | Iniciar / pausar (na pausa: START continua, A reinicia a fase, B volta ao menu) |

## Jogo
- 10 fases com 3 chefes (ver tabelas acima), 6 tipos de inimigos e musica propria em cada fase.
- Moedas (50 = 1 vida), corações (recuperam HP, máx. conforme a dificuldade), estrelas verdes (1UP), checkpoints, bandeira de fim de fase.
- Dano com invulnerabilidade temporária (piscando), espinhos, abismos, vidas, Game Over, tela de vitória.
- Ágata, Maria Francisca, Maxwell e Miguel aparecem nas fases e "falam" na linha de baixo do HUD ao chegar perto.
- Seleção de fases (fases já liberadas na sessão) e músicas diferentes por fase.
- Texto sem acentos: a fonte padrão do SGDK só tem ASCII.

## Estrutura
```
makefile  build.bat  README.md
tests/ run_tests.sh stub/ stubimpl.c test.c   (testes no PC)
src/   main.c game.c/h player.c/h enemies.c/h boss.c/h collectibles.c/h collision.c/h levels.c/h levels_gen.c menus.c/h
res/   resources.res  sprites/ tiles/ backgrounds/ audio/   (gerados por tools/gerar_assets.py)
tools/ gerar_assets.py (sprites, tiles, musicas, efeitos)  gerar_fases.py (fases 5-8 e 10 -> src/levels_gen.c)
out/   rom.bin (após compilar)
```
Diferenças em relação à estrutura sugerida: o arquivo de recursos do SGDK atual é `res/resources.res` (não `.rc`);
o `resources.h` é **gerado pelo rescomp** (não existe `inc/resources.h` manual); foram acrescentados `menus.h`, `boss.c/h` e `levels_gen.c` (gerado).

## Como funciona (resumo técnico)
- **Mapa:** células de 16×16 px (2×2 tiles) em `lvMap[]`, montado por `levels_load()` a partir de trechos de chão, blocos e objetos.
- **Rolagem:** o plano A (64×32 tiles) recebe só a coluna/linha nova ao cruzar cada célula (`streamUpdate`), com rolagem
  horizontal e vertical reais; o plano B (colinas/nuvens) rola em parallax (1/4).
- **HUD:** plano WINDOW (2 linhas fixas no topo).
- **Física:** ponto fixo (1/256 px), aceleração/atrito, gravidade, pulo variável, *coyote time*, plataformas sólidas e
  "de passagem", espinhos. Colisão por tiles em `collision.c`.
- **Sprites:** Davi 24×32 (parado, andando, correndo, pulando, caindo, dano, comemorando), inimigos, itens e Turma;
  sprites só existem enquanto estão perto da câmera.
- **Áudio:** driver XGM; músicas em VGM (PSG) e efeitos PCM 8 bits/14 kHz.

## Adicionar uma fase
1. Em `levels.c`, crie `Span[]`, `Block[]` e `Spawn[]` e uma entrada em `gLevels[]` (aumente `NUM_LEVELS` em `game.h`).
2. Se quiser outra música, gere mais um VGM e declare-o em `resources.res` e na tabela `songs[]` de `game.c`.
3. Regra de design usada: plataformas a no máximo 2 células de altura entre si; buracos de até 2–3 células.

## Solução de problemas
- **SGDK mais antigo (1.x):** `int main(bool hardReset)` → `int main()`; confira `SPR_initEx`, `XGM_*`, `Z80_loadDriver`.
- **Tiles bagunçados / sprites corrompidos:** ajuste `SPR_VRAM_TILES` em `game.h` (a reserva de VRAM do motor de sprites).
- **Erro do rescomp em WAV/XGM:** confirme que `xgmtool` do SGDK está presente; os VGM são só PSG.
- **Sem som de efeitos:** os IDs PCM usados são 64..72 (`SFX_FIRST_ID`).
