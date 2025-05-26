# PokemonCollection
Este projeto integra um banco de dados MySQL, Python com Power BI para análises detalhadas de cartas Pokémon do TCG, incluindo valores das cartas e simulação de combate.
📌 1️⃣ Estrutura do Banco de Dados
A tabela tbl_cards contém os seguintes atributos:
- name → Nome do Pokémon.
- hp → Pontos de vida.
- attack → Força de ataque.
- damage → Dano básico.
- weak → Fraqueza do Pokémon.
- resis → Resistência do Pokémon.
- type_id → Tipo do Pokémon (Fire, Water, etc.).
- stage_id → Estágio de evolução (Basic, Stage 1, Stage 2).
- image_url → URL da imagem da carta.

PontuacaoGeralCarta = 
VAR HP_Peso = AVERAGE(tbl_cards[hp]) * 0.4
VAR Ataque_Peso = AVERAGE(tbl_cards[attack]) * 0.5
VAR Bonus_Resistencia = IF(SELECTEDVALUE(tbl_cards[resis]) <> "None", 10, 0)
VAR Penalidade_Fraqueza = IF(SELECTEDVALUE(tbl_cards[weak]) <> "None", -10, 0)
RETURN HP_Peso + Ataque_Peso + Bonus_Resistencia + Penalidade_Fraqueza

🎯 1. Criando tabelas auxiliares para segmentação de Pokémon
TabelaPokemon1 = SELECTCOLUMNS(tbl_cards, "name", tbl_cards[name], "image_url", tbl_cards[image_url])

TabelaPokemon2 = SELECTCOLUMNS(tbl_cards, "name", tbl_cards[name], "image_url", tbl_cards[image_url])


✅ Finalidade: Criar tabelas auxiliares para permitir a seleção independente de Pokemon1 e Pokemon2.

🏆 2. Medida para calcular pontuação do combate
PontuacaoCombate = 
VAR Pokemon1 = SELECTEDVALUE(TabelaPokemon1[name])
VAR HP1 = LOOKUPVALUE(tbl_cards[hp], tbl_cards[name], Pokemon1)
VAR Dano1 = LOOKUPVALUE(tbl_cards[damage], tbl_cards[name], Pokemon1)
VAR Resist1 = IF(LOOKUPVALUE(tbl_cards[resis], tbl_cards[name], Pokemon1) <> "", 10, 0)
VAR Fraqueza1 = IF(LOOKUPVALUE(tbl_cards[weak], tbl_cards[name], Pokemon1) <> "", -10, 0)
VAR Retirada1 = LOOKUPVALUE(tbl_cards[retreat], tbl_cards[name], Pokemon1) * -5
VAR Estagio1 = SWITCH(LOOKUPVALUE(tbl_cards[stage_id], tbl_cards[name], Pokemon1), 1, 10, 2, 20, 0)
VAR Score1 = HP1 * 2 + Dano1 * 3 + Resist1 + Fraqueza1 + Retirada1 + Estagio1

VAR Pokemon2 = SELECTEDVALUE(TabelaPokemon2[name])
VAR HP2 = LOOKUPVALUE(tbl_cards[hp], tbl_cards[name], Pokemon2)
VAR Dano2 = LOOKUPVALUE(tbl_cards[damage], tbl_cards[name], Pokemon2)
VAR Resist2 = IF(LOOKUPVALUE(tbl_cards[resis], tbl_cards[name], Pokemon2) <> "", 10, 0)
VAR Fraqueza2 = IF(LOOKUPVALUE(tbl_cards[weak], tbl_cards[name], Pokemon2) <> "", -10, 0)
VAR Retirada2 = LOOKUPVALUE(tbl_cards[retreat], tbl_cards[name], Pokemon2) * -5
VAR Estagio2 = SWITCH(LOOKUPVALUE(tbl_cards[stage_id], tbl_cards[name], Pokemon2), 1, 10, 2, 20, 0)
VAR Score2 = HP2 * 2 + Dano2 * 3 + Resist2 + Fraqueza2 + Retirada2 + Estagio2

RETURN IF(Score1 > Score2, Pokemon1 & " Venceu!", Pokemon2 & " Venceu!")


✅ Finalidade: Calcula a pontuação de cada Pokémon com base em atributos e define o vencedor do combate.


📊 3. Normalização da pontuação para evitar discrepâncias
PontuacaoNormalizada = 
VAR MinPontuacao = CALCULATE(MINX(tbl_cards, [PontuacaoCombate]))
VAR MaxPontuacao = CALCULATE(MAXX(tbl_cards, [PontuacaoCombate]))
RETURN
    DIVIDE([PontuacaoCombate] - MinPontuacao, MaxPontuacao - MinPontuacao, 0)


✅ Finalidade: Evita valores discrepantes, normalizando pontuação entre 0 e 1 para comparação mais equilibrada.

Para atualizar as cartas no painel foi necessario criar um servidor local para buscar as imagens da cartas na internet.

