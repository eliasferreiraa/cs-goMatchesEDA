# Elias Ferreira de Faria 6 ° ADS
# Tema: CS:GO Professional Matches Analysis

### Background: Os dados usados neste notebook foram coletados em https://www.hltv.org/results. Foi raspado usando as bibliotecas 'requests' e 'BeautifulSoup'.
### Sobre o jogo:
**O Counter Strike Global Offensive, também conhecido como CS:GO, é um jogo de tiro em primeira pessoa (FPS) em que duas equipes, os terroristas e os contra-terroristas, se enfrentam em uma série de rodadas. Cada rodada tem um objetivo diferente, como plantar ou desarmar uma bomba, resgatar reféns ou eliminar toda a equipe adversária.**

**Os jogadores começam cada rodada com uma quantia limitada de dinheiro, que podem ser usados para comprar armas, equipamentos e granadas. As armas variam em termos de dano, alcance e cadência de tiro, e as granadas têm diferentes efeitos, como cegar os adversários, bloquear áreas ou causar dano.**

**Além disso, os jogadores podem comprar coletes e capacetes, que reduzem o dano que recebem, e kits de desativação de bomba, que aceleram o processo de desativação da bomba.
O objetivo dos terroristas varia dependendo do mapa, mas geralmente envolve plantar uma bomba em um dos pontos de bomba disponíveis no mapa. Os contra-terroristas devem impedir que isso aconteça, desarmar a bomba caso ela seja plantada ou eliminar todos os terroristas antes que eles tenham a chance de plantar a bomba.**

**Já o objetivo dos contra-terroristas pode variar entre resgatar reféns, proteger áreas do mapa ou simplesmente eliminar todos os terroristas.
O jogo é jogado em rodadas, e cada rodada tem uma duração máxima de dois minutos. A equipe vencedora de uma rodada ganha dinheiro extra, enquanto a equipe perdedora ganha menos dinheiro. Os jogadores também podem ganhar dinheiro ao completar objetivos ou matar inimigos.**

**No final de cada rodada, os jogadores podem usar o dinheiro ganho para comprar mais equipamentos, melhorar suas armas ou salvar dinheiro para rodadas futuras.**



# Preparação

**Importando bibliotecas e carregando as tabelas.**


```python
import numpy as np
import pandas as pd
import plotly.graph_objects as go
import plotly.express as px
from os import listdir

pd.set_option('display.max_columns',100)

listdir('C:\\Users\\PC4\\Jupyter\\csgo-professional-matches\\')
```




    ['economy.csv', 'picks.csv', 'players.csv', 'results.csv']




```python
base_dir = 'C:\\Users\\PC4\\Jupyter\\csgo-professional-matches\\'

results_df = pd.read_csv(base_dir+'results.csv',low_memory=False)
picks_df = pd.read_csv(base_dir+'picks.csv',low_memory=False)
economy_df = pd.read_csv(base_dir+'economy.csv',low_memory=False)
players_df = pd.read_csv(base_dir+'players.csv',low_memory=False)
```

# As primeiras linhas de cada tabela são apresentadas a seguir. Os dados são divididos em 4 tabelas que armazenam dados relacionados a:

**results_df:** pontuações do mapa e classificação das equipes;

**picks_df:** ordem de picks e vetos no processo de seleção de mapas;

**economic_df:** valor do equipamento inicial da rodada para todas as rodadas jogadas;

**players_df:** performances individuais dos jogadores em cada mapa.

**Os valores armazenados nas colunas 'event_id' e 'match_id' são exclusivos para cada correspondência e evento e compartilhados entre dataframes, portanto, essas colunas podem ser usadas como chaves para mesclar dados entre dataframes.**

**É necessário observar que as linhas em 'results_df' e 'economy_df' armazenam dados para cada mapa jogado em uma partida, enquanto as linhas em 'picks_df' e 'players_df' armazenam dados para toda a partida.**


```python
results_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>team_1</th>
      <th>team_2</th>
      <th>_map</th>
      <th>result_1</th>
      <th>result_2</th>
      <th>map_winner</th>
      <th>starting_ct</th>
      <th>ct_1</th>
      <th>t_2</th>
      <th>t_1</th>
      <th>ct_2</th>
      <th>event_id</th>
      <th>match_id</th>
      <th>rank_1</th>
      <th>rank_2</th>
      <th>map_wins_1</th>
      <th>map_wins_2</th>
      <th>match_winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-03-18</td>
      <td>Recon 5</td>
      <td>TeamOne</td>
      <td>Dust2</td>
      <td>0</td>
      <td>16</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>15</td>
      <td>5151</td>
      <td>2340454</td>
      <td>62</td>
      <td>63</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-03-18</td>
      <td>Recon 5</td>
      <td>TeamOne</td>
      <td>Inferno</td>
      <td>13</td>
      <td>16</td>
      <td>2</td>
      <td>2</td>
      <td>8</td>
      <td>6</td>
      <td>5</td>
      <td>10</td>
      <td>5151</td>
      <td>2340454</td>
      <td>62</td>
      <td>63</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-03-18</td>
      <td>New England Whalers</td>
      <td>Station7</td>
      <td>Inferno</td>
      <td>12</td>
      <td>16</td>
      <td>2</td>
      <td>1</td>
      <td>9</td>
      <td>6</td>
      <td>3</td>
      <td>10</td>
      <td>5243</td>
      <td>2340461</td>
      <td>140</td>
      <td>118</td>
      <td>12</td>
      <td>16</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-03-18</td>
      <td>Rugratz</td>
      <td>Bad News Bears</td>
      <td>Inferno</td>
      <td>7</td>
      <td>16</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>8</td>
      <td>7</td>
      <td>8</td>
      <td>5151</td>
      <td>2340453</td>
      <td>61</td>
      <td>38</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-03-18</td>
      <td>Rugratz</td>
      <td>Bad News Bears</td>
      <td>Vertigo</td>
      <td>8</td>
      <td>16</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
      <td>5</td>
      <td>4</td>
      <td>11</td>
      <td>5151</td>
      <td>2340453</td>
      <td>61</td>
      <td>38</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
picks_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>team_1</th>
      <th>team_2</th>
      <th>inverted_teams</th>
      <th>match_id</th>
      <th>event_id</th>
      <th>best_of</th>
      <th>system</th>
      <th>t1_removed_1</th>
      <th>t1_removed_2</th>
      <th>t1_removed_3</th>
      <th>t2_removed_1</th>
      <th>t2_removed_2</th>
      <th>t2_removed_3</th>
      <th>t1_picked_1</th>
      <th>t2_picked_1</th>
      <th>left_over</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-03-18</td>
      <td>TeamOne</td>
      <td>Recon 5</td>
      <td>1</td>
      <td>2340454</td>
      <td>5151</td>
      <td>3</td>
      <td>123412</td>
      <td>Vertigo</td>
      <td>Train</td>
      <td>0.0</td>
      <td>Nuke</td>
      <td>Overpass</td>
      <td>0.0</td>
      <td>Dust2</td>
      <td>Inferno</td>
      <td>Mirage</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-03-18</td>
      <td>Rugratz</td>
      <td>Bad News Bears</td>
      <td>0</td>
      <td>2340453</td>
      <td>5151</td>
      <td>3</td>
      <td>123412</td>
      <td>Dust2</td>
      <td>Nuke</td>
      <td>0.0</td>
      <td>Mirage</td>
      <td>Train</td>
      <td>0.0</td>
      <td>Vertigo</td>
      <td>Inferno</td>
      <td>Overpass</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-03-18</td>
      <td>New England Whalers</td>
      <td>Station7</td>
      <td>0</td>
      <td>2340461</td>
      <td>5243</td>
      <td>1</td>
      <td>121212</td>
      <td>Mirage</td>
      <td>Dust2</td>
      <td>Vertigo</td>
      <td>Nuke</td>
      <td>Train</td>
      <td>Overpass</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>Inferno</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-03-17</td>
      <td>Complexity</td>
      <td>forZe</td>
      <td>1</td>
      <td>2340279</td>
      <td>5226</td>
      <td>3</td>
      <td>123412</td>
      <td>Inferno</td>
      <td>Nuke</td>
      <td>0.0</td>
      <td>Overpass</td>
      <td>Vertigo</td>
      <td>0.0</td>
      <td>Dust2</td>
      <td>Train</td>
      <td>Mirage</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-03-17</td>
      <td>Singularity</td>
      <td>Endpoint</td>
      <td>0</td>
      <td>2340456</td>
      <td>5247</td>
      <td>3</td>
      <td>123412</td>
      <td>Train</td>
      <td>Mirage</td>
      <td>0.0</td>
      <td>Nuke</td>
      <td>Inferno</td>
      <td>0.0</td>
      <td>Overpass</td>
      <td>Vertigo</td>
      <td>Dust2</td>
    </tr>
  </tbody>
</table>
</div>




```python
economy_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>match_id</th>
      <th>event_id</th>
      <th>team_1</th>
      <th>team_2</th>
      <th>best_of</th>
      <th>_map</th>
      <th>t1_start</th>
      <th>t2_start</th>
      <th>1_t1</th>
      <th>2_t1</th>
      <th>3_t1</th>
      <th>4_t1</th>
      <th>5_t1</th>
      <th>6_t1</th>
      <th>7_t1</th>
      <th>8_t1</th>
      <th>9_t1</th>
      <th>10_t1</th>
      <th>11_t1</th>
      <th>12_t1</th>
      <th>13_t1</th>
      <th>14_t1</th>
      <th>15_t1</th>
      <th>16_t1</th>
      <th>17_t1</th>
      <th>18_t1</th>
      <th>19_t1</th>
      <th>20_t1</th>
      <th>21_t1</th>
      <th>22_t1</th>
      <th>23_t1</th>
      <th>24_t1</th>
      <th>25_t1</th>
      <th>26_t1</th>
      <th>27_t1</th>
      <th>28_t1</th>
      <th>29_t1</th>
      <th>30_t1</th>
      <th>1_t2</th>
      <th>2_t2</th>
      <th>3_t2</th>
      <th>4_t2</th>
      <th>5_t2</th>
      <th>6_t2</th>
      <th>7_t2</th>
      <th>8_t2</th>
      <th>9_t2</th>
      <th>10_t2</th>
      <th>11_t2</th>
      <th>12_t2</th>
      <th>13_t2</th>
      <th>14_t2</th>
      <th>15_t2</th>
      <th>16_t2</th>
      <th>17_t2</th>
      <th>18_t2</th>
      <th>19_t2</th>
      <th>20_t2</th>
      <th>21_t2</th>
      <th>22_t2</th>
      <th>23_t2</th>
      <th>24_t2</th>
      <th>25_t2</th>
      <th>26_t2</th>
      <th>27_t2</th>
      <th>28_t2</th>
      <th>29_t2</th>
      <th>30_t2</th>
      <th>1_winner</th>
      <th>2_winner</th>
      <th>3_winner</th>
      <th>4_winner</th>
      <th>5_winner</th>
      <th>6_winner</th>
      <th>7_winner</th>
      <th>8_winner</th>
      <th>9_winner</th>
      <th>10_winner</th>
      <th>11_winner</th>
      <th>12_winner</th>
      <th>13_winner</th>
      <th>14_winner</th>
      <th>15_winner</th>
      <th>16_winner</th>
      <th>17_winner</th>
      <th>18_winner</th>
      <th>19_winner</th>
      <th>20_winner</th>
      <th>21_winner</th>
      <th>22_winner</th>
      <th>23_winner</th>
      <th>24_winner</th>
      <th>25_winner</th>
      <th>26_winner</th>
      <th>27_winner</th>
      <th>28_winner</th>
      <th>29_winner</th>
      <th>30_winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-03-01</td>
      <td>2339402</td>
      <td>4901</td>
      <td>G2</td>
      <td>Natus Vincere</td>
      <td>5</td>
      <td>Nuke</td>
      <td>t</td>
      <td>ct</td>
      <td>4350.0</td>
      <td>1100.0</td>
      <td>22100.0</td>
      <td>9350.0</td>
      <td>25750.0</td>
      <td>10400.0</td>
      <td>24600.0</td>
      <td>8150.0</td>
      <td>26700.0</td>
      <td>23400.0</td>
      <td>4300.0</td>
      <td>25900.0</td>
      <td>11950.0</td>
      <td>24850.0</td>
      <td>21900.0</td>
      <td>4150.0</td>
      <td>10650.0</td>
      <td>27300.0</td>
      <td>27000.0</td>
      <td>30950.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4250.0</td>
      <td>20250.0</td>
      <td>24600.0</td>
      <td>29300.0</td>
      <td>30650.0</td>
      <td>31450.0</td>
      <td>32050.0</td>
      <td>31450.0</td>
      <td>32050.0</td>
      <td>30950.0</td>
      <td>29300.0</td>
      <td>30200.0</td>
      <td>31550.0</td>
      <td>32350.0</td>
      <td>33050.0</td>
      <td>4250.0</td>
      <td>20300.0</td>
      <td>8250.0</td>
      <td>1300.0</td>
      <td>22200.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-03-01</td>
      <td>2339402</td>
      <td>4901</td>
      <td>G2</td>
      <td>Natus Vincere</td>
      <td>5</td>
      <td>Dust2</td>
      <td>ct</td>
      <td>t</td>
      <td>3900.0</td>
      <td>7400.0</td>
      <td>23250.0</td>
      <td>28500.0</td>
      <td>31900.0</td>
      <td>31700.0</td>
      <td>18950.0</td>
      <td>30200.0</td>
      <td>28650.0</td>
      <td>30350.0</td>
      <td>30150.0</td>
      <td>11100.0</td>
      <td>23700.0</td>
      <td>8550.0</td>
      <td>26350.0</td>
      <td>4050.0</td>
      <td>9400.0</td>
      <td>21900.0</td>
      <td>12700.0</td>
      <td>11300.0</td>
      <td>3100.0</td>
      <td>26250.0</td>
      <td>21300.0</td>
      <td>23950.0</td>
      <td>27450.0</td>
      <td>27550.0</td>
      <td>28050.0</td>
      <td>26250.0</td>
      <td>26250.0</td>
      <td>NaN</td>
      <td>3500.0</td>
      <td>18550.0</td>
      <td>7300.0</td>
      <td>1500.0</td>
      <td>21800.0</td>
      <td>12000.0</td>
      <td>21050.0</td>
      <td>24450.0</td>
      <td>6850.0</td>
      <td>26850.0</td>
      <td>23100.0</td>
      <td>25650.0</td>
      <td>26800.0</td>
      <td>26750.0</td>
      <td>28250.0</td>
      <td>4000.0</td>
      <td>18850.0</td>
      <td>15850.0</td>
      <td>23000.0</td>
      <td>26850.0</td>
      <td>29100.0</td>
      <td>26300.0</td>
      <td>26850.0</td>
      <td>19050.0</td>
      <td>3500.0</td>
      <td>26450.0</td>
      <td>27450.0</td>
      <td>27500.0</td>
      <td>29050.0</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-03-01</td>
      <td>2339402</td>
      <td>4901</td>
      <td>G2</td>
      <td>Natus Vincere</td>
      <td>5</td>
      <td>Mirage</td>
      <td>t</td>
      <td>ct</td>
      <td>4150.0</td>
      <td>14300.0</td>
      <td>2000.0</td>
      <td>24800.0</td>
      <td>9000.0</td>
      <td>23150.0</td>
      <td>21850.0</td>
      <td>23700.0</td>
      <td>10450.0</td>
      <td>26250.0</td>
      <td>8800.0</td>
      <td>24950.0</td>
      <td>12100.0</td>
      <td>24350.0</td>
      <td>18250.0</td>
      <td>4300.0</td>
      <td>19400.0</td>
      <td>8900.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4200.0</td>
      <td>21200.0</td>
      <td>24150.0</td>
      <td>27050.0</td>
      <td>31350.0</td>
      <td>30650.0</td>
      <td>31050.0</td>
      <td>28150.0</td>
      <td>29650.0</td>
      <td>30950.0</td>
      <td>31550.0</td>
      <td>32950.0</td>
      <td>32250.0</td>
      <td>31650.0</td>
      <td>33350.0</td>
      <td>4350.0</td>
      <td>14300.0</td>
      <td>22850.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-02-29</td>
      <td>2339401</td>
      <td>4901</td>
      <td>Natus Vincere</td>
      <td>Astralis</td>
      <td>3</td>
      <td>Dust2</td>
      <td>t</td>
      <td>ct</td>
      <td>4150.0</td>
      <td>18050.0</td>
      <td>21000.0</td>
      <td>25850.0</td>
      <td>25000.0</td>
      <td>25000.0</td>
      <td>27250.0</td>
      <td>26150.0</td>
      <td>26300.0</td>
      <td>27850.0</td>
      <td>26750.0</td>
      <td>27450.0</td>
      <td>27850.0</td>
      <td>18300.0</td>
      <td>27850.0</td>
      <td>4000.0</td>
      <td>21100.0</td>
      <td>9100.0</td>
      <td>3100.0</td>
      <td>22100.0</td>
      <td>23350.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4250.0</td>
      <td>9300.0</td>
      <td>2100.0</td>
      <td>19300.0</td>
      <td>27250.0</td>
      <td>9850.0</td>
      <td>24800.0</td>
      <td>12200.0</td>
      <td>27550.0</td>
      <td>26100.0</td>
      <td>7450.0</td>
      <td>27650.0</td>
      <td>24500.0</td>
      <td>19100.0</td>
      <td>18050.0</td>
      <td>4150.0</td>
      <td>15100.0</td>
      <td>18450.0</td>
      <td>23150.0</td>
      <td>28150.0</td>
      <td>27850.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-02-29</td>
      <td>2339401</td>
      <td>4901</td>
      <td>Natus Vincere</td>
      <td>Astralis</td>
      <td>3</td>
      <td>Nuke</td>
      <td>ct</td>
      <td>t</td>
      <td>4200.0</td>
      <td>10000.0</td>
      <td>22000.0</td>
      <td>24500.0</td>
      <td>27550.0</td>
      <td>29350.0</td>
      <td>31950.0</td>
      <td>31850.0</td>
      <td>31750.0</td>
      <td>32850.0</td>
      <td>32150.0</td>
      <td>31750.0</td>
      <td>31850.0</td>
      <td>33050.0</td>
      <td>33150.0</td>
      <td>4250.0</td>
      <td>3000.0</td>
      <td>21150.0</td>
      <td>11750.0</td>
      <td>28050.0</td>
      <td>25900.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4200.0</td>
      <td>17950.0</td>
      <td>9050.0</td>
      <td>1000.0</td>
      <td>22850.0</td>
      <td>7500.0</td>
      <td>24400.0</td>
      <td>24700.0</td>
      <td>9700.0</td>
      <td>27150.0</td>
      <td>21850.0</td>
      <td>21900.0</td>
      <td>8150.0</td>
      <td>25200.0</td>
      <td>20450.0</td>
      <td>4300.0</td>
      <td>19300.0</td>
      <td>27400.0</td>
      <td>31700.0</td>
      <td>29650.0</td>
      <td>12150.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
players_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>player_name</th>
      <th>team</th>
      <th>opponent</th>
      <th>country</th>
      <th>player_id</th>
      <th>match_id</th>
      <th>event_id</th>
      <th>event_name</th>
      <th>best_of</th>
      <th>map_1</th>
      <th>map_2</th>
      <th>map_3</th>
      <th>kills</th>
      <th>assists</th>
      <th>deaths</th>
      <th>hs</th>
      <th>flash_assists</th>
      <th>kast</th>
      <th>kddiff</th>
      <th>adr</th>
      <th>fkdiff</th>
      <th>rating</th>
      <th>m1_kills</th>
      <th>m1_assists</th>
      <th>m1_deaths</th>
      <th>m1_hs</th>
      <th>m1_flash_assists</th>
      <th>m1_kast</th>
      <th>m1_kddiff</th>
      <th>m1_adr</th>
      <th>m1_fkdiff</th>
      <th>m1_rating</th>
      <th>m2_kills</th>
      <th>m2_assists</th>
      <th>m2_deaths</th>
      <th>m2_hs</th>
      <th>m2_flash_assists</th>
      <th>m2_kast</th>
      <th>m2_kddiff</th>
      <th>m2_adr</th>
      <th>m2_fkdiff</th>
      <th>m2_rating</th>
      <th>m3_kills</th>
      <th>m3_assists</th>
      <th>m3_deaths</th>
      <th>m3_hs</th>
      <th>m3_flash_assists</th>
      <th>m3_kast</th>
      <th>m3_kddiff</th>
      <th>...</th>
      <th>m3_fkdiff</th>
      <th>m3_rating</th>
      <th>kills_ct</th>
      <th>deaths_ct</th>
      <th>kddiff_ct</th>
      <th>adr_ct</th>
      <th>kast_ct</th>
      <th>rating_ct</th>
      <th>kills_t</th>
      <th>deaths_t</th>
      <th>kddiff_t</th>
      <th>adr_t</th>
      <th>kast_t</th>
      <th>rating_t</th>
      <th>m1_kills_ct</th>
      <th>m1_deaths_ct</th>
      <th>m1_kddiff_ct</th>
      <th>m1_adr_ct</th>
      <th>m1_kast_ct</th>
      <th>m1_rating_ct</th>
      <th>m1_kills_t</th>
      <th>m1_deaths_t</th>
      <th>m1_kddiff_t</th>
      <th>m1_adr_t</th>
      <th>m1_kast_t</th>
      <th>m1_rating_t</th>
      <th>m2_kills_ct</th>
      <th>m2_deaths_ct</th>
      <th>m2_kddiff_ct</th>
      <th>m2_adr_ct</th>
      <th>m2_kast_ct</th>
      <th>m2_rating_ct</th>
      <th>m2_kills_t</th>
      <th>m2_deaths_t</th>
      <th>m2_kddiff_t</th>
      <th>m2_adr_t</th>
      <th>m2_kast_t</th>
      <th>m2_rating_t</th>
      <th>m3_kills_ct</th>
      <th>m3_deaths_ct</th>
      <th>m3_kddiff_ct</th>
      <th>m3_adr_ct</th>
      <th>m3_kast_ct</th>
      <th>m3_rating_ct</th>
      <th>m3_kills_t</th>
      <th>m3_deaths_t</th>
      <th>m3_kddiff_t</th>
      <th>m3_adr_t</th>
      <th>m3_kast_t</th>
      <th>m3_rating_t</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-02-26</td>
      <td>Brehze</td>
      <td>Evil Geniuses</td>
      <td>Liquid</td>
      <td>United States</td>
      <td>9136</td>
      <td>2339385</td>
      <td>4901</td>
      <td>IEM Katowice 2020</td>
      <td>3</td>
      <td>Overpass</td>
      <td>Nuke</td>
      <td>Inferno</td>
      <td>57</td>
      <td>14</td>
      <td>61</td>
      <td>29</td>
      <td>0.0</td>
      <td>71.1</td>
      <td>-4</td>
      <td>79.9</td>
      <td>0</td>
      <td>1.04</td>
      <td>11</td>
      <td>3</td>
      <td>18</td>
      <td>5</td>
      <td>0.0</td>
      <td>65.2</td>
      <td>-7</td>
      <td>60.8</td>
      <td>-1</td>
      <td>0.70</td>
      <td>30.0</td>
      <td>7.0</td>
      <td>24.0</td>
      <td>16.0</td>
      <td>0.0</td>
      <td>73.5</td>
      <td>6.0</td>
      <td>99.2</td>
      <td>6.0</td>
      <td>1.38</td>
      <td>16.0</td>
      <td>4.0</td>
      <td>19.0</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>73.1</td>
      <td>-3.0</td>
      <td>...</td>
      <td>-5.0</td>
      <td>0.91</td>
      <td>34.0</td>
      <td>30.0</td>
      <td>4.0</td>
      <td>81.6</td>
      <td>79.2</td>
      <td>1.10</td>
      <td>23.0</td>
      <td>31.0</td>
      <td>-8.0</td>
      <td>77.5</td>
      <td>60.0</td>
      <td>0.97</td>
      <td>8.0</td>
      <td>10.0</td>
      <td>-2.0</td>
      <td>76.3</td>
      <td>73.3</td>
      <td>0.90</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>-5.0</td>
      <td>31.9</td>
      <td>50.0</td>
      <td>0.34</td>
      <td>17.0</td>
      <td>10.0</td>
      <td>7.0</td>
      <td>93.7</td>
      <td>83.3</td>
      <td>1.41</td>
      <td>13.0</td>
      <td>14.0</td>
      <td>-1.0</td>
      <td>105.3</td>
      <td>62.5</td>
      <td>1.35</td>
      <td>9.0</td>
      <td>10.0</td>
      <td>-1.0</td>
      <td>72.5</td>
      <td>80.0</td>
      <td>0.93</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>-2.0</td>
      <td>70.4</td>
      <td>63.6</td>
      <td>0.89</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-02-26</td>
      <td>CeRq</td>
      <td>Evil Geniuses</td>
      <td>Liquid</td>
      <td>Bulgaria</td>
      <td>11219</td>
      <td>2339385</td>
      <td>4901</td>
      <td>IEM Katowice 2020</td>
      <td>3</td>
      <td>Overpass</td>
      <td>Nuke</td>
      <td>Inferno</td>
      <td>54</td>
      <td>10</td>
      <td>54</td>
      <td>18</td>
      <td>4.0</td>
      <td>65.1</td>
      <td>0</td>
      <td>71.7</td>
      <td>2</td>
      <td>0.98</td>
      <td>11</td>
      <td>2</td>
      <td>17</td>
      <td>4</td>
      <td>2.0</td>
      <td>60.9</td>
      <td>-6</td>
      <td>68.9</td>
      <td>-1</td>
      <td>0.75</td>
      <td>26.0</td>
      <td>6.0</td>
      <td>19.0</td>
      <td>8.0</td>
      <td>1.0</td>
      <td>76.5</td>
      <td>7.0</td>
      <td>80.1</td>
      <td>3.0</td>
      <td>1.24</td>
      <td>17.0</td>
      <td>2.0</td>
      <td>18.0</td>
      <td>6.0</td>
      <td>1.0</td>
      <td>53.8</td>
      <td>-1.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.87</td>
      <td>37.0</td>
      <td>25.0</td>
      <td>12.0</td>
      <td>77.4</td>
      <td>72.9</td>
      <td>1.16</td>
      <td>17.0</td>
      <td>29.0</td>
      <td>-12.0</td>
      <td>63.9</td>
      <td>54.3</td>
      <td>0.73</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>0.0</td>
      <td>72.3</td>
      <td>73.3</td>
      <td>0.88</td>
      <td>2.0</td>
      <td>8.0</td>
      <td>-6.0</td>
      <td>62.4</td>
      <td>37.5</td>
      <td>0.50</td>
      <td>15.0</td>
      <td>6.0</td>
      <td>9.0</td>
      <td>79.8</td>
      <td>88.9</td>
      <td>1.45</td>
      <td>11.0</td>
      <td>13.0</td>
      <td>-2.0</td>
      <td>80.5</td>
      <td>62.5</td>
      <td>1.00</td>
      <td>13.0</td>
      <td>10.0</td>
      <td>3.0</td>
      <td>79.5</td>
      <td>53.3</td>
      <td>1.12</td>
      <td>4.0</td>
      <td>8.0</td>
      <td>-4.0</td>
      <td>40.7</td>
      <td>54.5</td>
      <td>0.53</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-02-26</td>
      <td>EliGE</td>
      <td>Liquid</td>
      <td>Evil Geniuses</td>
      <td>United States</td>
      <td>8738</td>
      <td>2339385</td>
      <td>4901</td>
      <td>IEM Katowice 2020</td>
      <td>3</td>
      <td>Overpass</td>
      <td>Nuke</td>
      <td>Inferno</td>
      <td>55</td>
      <td>10</td>
      <td>51</td>
      <td>28</td>
      <td>1.0</td>
      <td>67.5</td>
      <td>4</td>
      <td>77.9</td>
      <td>1</td>
      <td>1.08</td>
      <td>15</td>
      <td>3</td>
      <td>12</td>
      <td>9</td>
      <td>0.0</td>
      <td>69.6</td>
      <td>3</td>
      <td>77.0</td>
      <td>3</td>
      <td>1.32</td>
      <td>24.0</td>
      <td>3.0</td>
      <td>24.0</td>
      <td>14.0</td>
      <td>0.0</td>
      <td>64.7</td>
      <td>0.0</td>
      <td>72.9</td>
      <td>-1.0</td>
      <td>0.97</td>
      <td>16.0</td>
      <td>4.0</td>
      <td>15.0</td>
      <td>5.0</td>
      <td>1.0</td>
      <td>69.2</td>
      <td>1.0</td>
      <td>...</td>
      <td>-1.0</td>
      <td>1.04</td>
      <td>31.0</td>
      <td>17.0</td>
      <td>14.0</td>
      <td>96.6</td>
      <td>71.4</td>
      <td>1.39</td>
      <td>24.0</td>
      <td>34.0</td>
      <td>-10.0</td>
      <td>64.2</td>
      <td>64.6</td>
      <td>0.86</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>135.2</td>
      <td>75.0</td>
      <td>2.17</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>-6.0</td>
      <td>45.9</td>
      <td>66.7</td>
      <td>0.87</td>
      <td>13.0</td>
      <td>9.0</td>
      <td>4.0</td>
      <td>87.6</td>
      <td>75.0</td>
      <td>1.26</td>
      <td>11.0</td>
      <td>15.0</td>
      <td>-4.0</td>
      <td>59.7</td>
      <td>55.6</td>
      <td>0.71</td>
      <td>7.0</td>
      <td>6.0</td>
      <td>1.0</td>
      <td>81.5</td>
      <td>63.6</td>
      <td>1.03</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>0.0</td>
      <td>87.9</td>
      <td>73.3</td>
      <td>1.05</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-02-26</td>
      <td>Ethan</td>
      <td>Evil Geniuses</td>
      <td>Liquid</td>
      <td>United States</td>
      <td>10671</td>
      <td>2339385</td>
      <td>4901</td>
      <td>IEM Katowice 2020</td>
      <td>3</td>
      <td>Overpass</td>
      <td>Nuke</td>
      <td>Inferno</td>
      <td>43</td>
      <td>5</td>
      <td>54</td>
      <td>18</td>
      <td>2.0</td>
      <td>65.1</td>
      <td>-11</td>
      <td>58.7</td>
      <td>-4</td>
      <td>0.83</td>
      <td>11</td>
      <td>1</td>
      <td>15</td>
      <td>6</td>
      <td>1.0</td>
      <td>65.2</td>
      <td>-4</td>
      <td>60.7</td>
      <td>-2</td>
      <td>0.73</td>
      <td>22.0</td>
      <td>3.0</td>
      <td>21.0</td>
      <td>8.0</td>
      <td>1.0</td>
      <td>70.6</td>
      <td>1.0</td>
      <td>67.9</td>
      <td>-2.0</td>
      <td>1.00</td>
      <td>10.0</td>
      <td>1.0</td>
      <td>18.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>57.7</td>
      <td>-8.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.69</td>
      <td>33.0</td>
      <td>23.0</td>
      <td>10.0</td>
      <td>74.0</td>
      <td>75.0</td>
      <td>1.11</td>
      <td>10.0</td>
      <td>31.0</td>
      <td>-21.0</td>
      <td>37.8</td>
      <td>51.4</td>
      <td>0.43</td>
      <td>9.0</td>
      <td>8.0</td>
      <td>1.0</td>
      <td>68.3</td>
      <td>73.3</td>
      <td>0.92</td>
      <td>2.0</td>
      <td>7.0</td>
      <td>-5.0</td>
      <td>46.5</td>
      <td>50.0</td>
      <td>0.38</td>
      <td>15.0</td>
      <td>6.0</td>
      <td>9.0</td>
      <td>84.3</td>
      <td>83.3</td>
      <td>1.40</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>-8.0</td>
      <td>49.3</td>
      <td>56.2</td>
      <td>0.55</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>0.0</td>
      <td>67.2</td>
      <td>66.7</td>
      <td>0.97</td>
      <td>1.0</td>
      <td>9.0</td>
      <td>-8.0</td>
      <td>14.8</td>
      <td>45.5</td>
      <td>0.31</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-02-26</td>
      <td>NAF</td>
      <td>Liquid</td>
      <td>Evil Geniuses</td>
      <td>Canada</td>
      <td>8520</td>
      <td>2339385</td>
      <td>4901</td>
      <td>IEM Katowice 2020</td>
      <td>3</td>
      <td>Overpass</td>
      <td>Nuke</td>
      <td>Inferno</td>
      <td>52</td>
      <td>22</td>
      <td>46</td>
      <td>23</td>
      <td>9.0</td>
      <td>77.1</td>
      <td>6</td>
      <td>75.9</td>
      <td>-1</td>
      <td>1.08</td>
      <td>10</td>
      <td>5</td>
      <td>12</td>
      <td>3</td>
      <td>3.0</td>
      <td>65.2</td>
      <td>-2</td>
      <td>51.5</td>
      <td>0</td>
      <td>0.83</td>
      <td>29.0</td>
      <td>6.0</td>
      <td>21.0</td>
      <td>16.0</td>
      <td>0.0</td>
      <td>82.4</td>
      <td>8.0</td>
      <td>101.9</td>
      <td>0.0</td>
      <td>1.35</td>
      <td>13.0</td>
      <td>11.0</td>
      <td>13.0</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>80.8</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.0</td>
      <td>0.98</td>
      <td>28.0</td>
      <td>17.0</td>
      <td>11.0</td>
      <td>96.3</td>
      <td>85.7</td>
      <td>1.36</td>
      <td>24.0</td>
      <td>29.0</td>
      <td>-5.0</td>
      <td>61.0</td>
      <td>70.8</td>
      <td>0.87</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>64.8</td>
      <td>62.5</td>
      <td>0.98</td>
      <td>6.0</td>
      <td>9.0</td>
      <td>-3.0</td>
      <td>44.4</td>
      <td>66.7</td>
      <td>0.75</td>
      <td>19.0</td>
      <td>8.0</td>
      <td>11.0</td>
      <td>128.1</td>
      <td>100.0</td>
      <td>1.88</td>
      <td>10.0</td>
      <td>13.0</td>
      <td>-3.0</td>
      <td>78.7</td>
      <td>66.7</td>
      <td>0.89</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>-1.0</td>
      <td>72.9</td>
      <td>81.8</td>
      <td>0.96</td>
      <td>8.0</td>
      <td>7.0</td>
      <td>1.0</td>
      <td>56.3</td>
      <td>80.0</td>
      <td>0.99</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 101 columns</p>
</div>



**Os dados coletados contêm dados de todas as partidas profissionais de CS:GO, incluindo partidas de times relativamente desconhecidos. Por esse motivo, limitaremos os conjuntos de dados às partidas disputadas entre os 30 melhores times do ranking HLTV.**


```python
min_rank = 30
results_df = results_df[(results_df.rank_1<min_rank)&(results_df.rank_2<min_rank)]

picks_df     = picks_df  [picks_df  .match_id.isin(results_df.match_id.unique())]
economy_df   = economy_df[economy_df.match_id.isin(results_df.match_id.unique())]
players_df   = players_df[players_df.match_id.isin(results_df.match_id.unique())]
```


```python
winner_1 = results_df[results_df.result_1>=results_df.result_2].result_1.values
loser_1  = results_df[results_df.result_1>=results_df.result_2].result_2.values

winner_2 = results_df[results_df.result_1<results_df.result_2].result_2.values
loser_2  = results_df[results_df.result_1<results_df.result_2].result_1.values

winner = np.concatenate((winner_1,winner_2))
loser = np.concatenate((loser_1,loser_2))
scores_df = pd.DataFrame(np.vstack((winner,loser)).T,columns=['winner','loser'])
```


```python
gb = scores_df.groupby(by=['winner','loser'])['winner'].count()/scores_df.shape[0]
overtime_percentage = str(round(gb[gb.index.get_level_values(0)!=16].sum()*100,1))+'%'

gb = round(gb[gb>10**-3]*100,1)

index_plot = np.array(gb.index.get_level_values(0).astype('str'))+'-'+np.array(
    gb.index.get_level_values(1).astype('str'))

fig = go.Figure()
fig.add_trace(go.Scatter(x=index_plot,y=gb.values, name='results'))
fig.update_layout(xaxis_type='category',title='Scores distribution',xaxis_title='Score',yaxis_title='Percentage of matches (%)')
```

```python
overtime_percentage
```

    '9.7%'


**Podemos observar que no tempo regulamentar (desconsiderando a prorrogação), o placar mais comum é 16-14 (obtido em 10,7% das partidas) e o mais raro é 16-0 (obtido em apenas 0,2% das partidas), com intermediários pontuações caindo em algum lugar no meio.**

**9,7% das partidas vão para a prorrogação.**

**Os resultados podem diferir se considerarmos partidas disputadas por times que não são de primeira linha.**

# Mapa mais favorável aos CTs


**Há muito tempo existe uma disputa no CS:GO para determinar os mapas mais favoráveis aos CTs, e sempre há discussão se ter um mapa muito desequilibrado é um resultado desejável. Aqui determinamos essa característica calculando as pontuações médias obtidas em cada lado do mapa e depois comparando ambos os lados.**


```python
ct_1 = results_df[['date','_map','ct_1']].rename(columns={'ct_1':'ct'})
ct_2 = results_df[['date','_map','ct_2']].rename(columns={'ct_2':'ct'})
ct = pd.concat((ct_1,ct_2))
```


```python
t_1 = results_df[['date','_map','t_1']].rename(columns={'t_1':'t'})
t_2 = results_df[['date','_map','t_2']].rename(columns={'t_2':'t'})
t = pd.concat((t_1,t_2))
```


```python
t = t.sort_values('date')
ct = ct.sort_values('date')
```


```python
maps = ['Cache','Cobblestone','Dust2','Inferno','Mirage','Nuke','Overpass','Train','Vertigo']
```


```python
series_t, series_ct, how_ct = {},{},{}
for i, key in enumerate(maps):
    t_map = t[t._map == maps[i]]
    ct_map = ct[ct._map == maps[i]]
    y_t = t_map.t.rolling(min_periods = 20, window= 200, center=True).sum().values
    y_ct = ct_map.ct.rolling(min_periods = 20, window= 200, center=True).sum().values
    
    series_t[key] = pd.Series(data=y_t,index=t_map.date)
    series_ct[key] = pd.Series(data=y_ct,index=ct_map.date)
    
    how_ct[key] = series_ct[key]/(series_ct[key]+series_t[key])//0.001/10
```


```python
def add_trace(_map):
    fig.add_trace(go.Scatter(x=how_ct[_map].index, y=how_ct[_map].values, name=_map))
```


```python
fig = go.Figure()
for _map in maps:
    add_trace(_map)
fig.add_trace(go.Scatter(x=['2015-11-01', '2020-03-12'], y=[50,50],
                         mode='lines',line=dict(color='grey'),showlegend=False))
fig.update_layout(title='Distribution of rounds between CT and T sides',
                  yaxis_title='Percentage of round won on the CT-side (%)')
fig.show()
```

**Existem longos períodos sem dados para um mapa no gráfico. Isso ocorre porque os mapas são adicionados e removidos constantemente da lista de mapas disponíveis.**

**Nuke e Train oscilam como sendo os mapas mais favoráveis aos CTs, tendo cerca de 57% das rodadas vencidas pelos CTs, enquanto Dust2 e Cache são historicamente os mapas mais favoráveis aos Terroristas (T).**

**É interessante notar que Inferno era conhecido por ser um mapa muito favorável aos CTs antes de 2016, o que foi uma das razões para atualizá-lo. Desde a sua atualização, Inferno tem sido o mapa mais equilibrado nesse aspecto.**

# Mapas jogados por período

## Sobre os mapas:

**Mirage, Train, Inferno e Overpass são os mapas dos quais temos mais dados disponíveis. Eles também são os mapas presentes no mapa de jogo por mais tempo;**

**Cache, Cobblestone e Dust2 foram jogados menos vezes, mas também estiveram fora do mapa de jogo por longos períodos:**

**Nuke é historicamente o mapa menos jogado, mesmo estando presente no mapa de jogo por muito tempo. A única explicação para isso vem da falta de familiaridade das equipes com o mapa;**

**Vertigo tem dados limitados disponíveis, pois foi o mapa mais recentemente adicionado ao mapa de jogo.**


```python
print('Total number of matches played on the map:')
results_df.groupby('_map').date.count()
```

    Total number of matches played on the map:
    




    _map
    Cache           900
    Cobblestone     898
    Default           2
    Dust2           892
    Inferno        1325
    Mirage         1617
    Nuke            750
    Overpass       1150
    Train          1384
    Vertigo          99
    Name: date, dtype: int64



**No CS:GO, os torneios mais respeitados são os Majors. Esses torneios são normalmente disputados duas vezes por ano e têm um prêmio de $1.000.000. Mais informações sobre o assunto podem ser vistas aqui: https://liquipedia.net/counterstrike/Majors**

**Para a próxima etapa, vamos discretizar a coluna 'data' em um dataframe em uma coluna 'time_period'. Essa nova coluna se referirá ao torneio Major mais recentemente disputado.**

**Como exemplo, seguindo essa técnica de binning, atualmente estamos no período de Berlim 2019, pois esse foi o torneio mais recentemente disputado.**


```python
majors = [{'tournament':'01. Cluj-Napoca 2015','start_date':'2015-10-28'},
          {'tournament':'02. Columbus 2016','start_date':'2016-03-29'},
          {'tournament':'03. Cologne 2016','start_date':'2016-07-05'},
          {'tournament':'04. Atlanta 2017','start_date':'2017-01-22'},
          {'tournament':'05. Krakow 2017','start_date':'2017-07-16'},
          {'tournament':'06. Boston 2018','start_date':'2018-01-26'},
          {'tournament':'07. London 2018','start_date':'2018-09-20'},
          {'tournament':'08. Katowice 2019','start_date':'2019-02-28'},
          {'tournament':'09. Berlin 2019','start_date':'2019-09-05'}]
```


```python
def create_col_time_period(df):
    df['time_period'] = ''
    
    for major_start in majors:
        df.loc[(df['date']>=major_start['start_date']),'time_period'] = major_start['tournament']
    
    return df
```


```python
results_df = create_col_time_period(results_df)
economy_df = create_col_time_period(economy_df)
picks_df = create_col_time_period(picks_df)
players_df = players_df.merge(results_df[['match_id','time_period']],'left',on='match_id')
```


```python
results_df_team_1 = results_df[['time_period','team_1','_map','ct_1','t_2','ct_2','t_1']
                      ].rename(columns={'team_1':'team'})
results_df_team_2 = results_df[['time_period','team_2','_map','ct_1','t_2','ct_2','t_1']
                      ].rename(columns={'team_2':'team'})
results_df_teams = pd.concat((results_df_team_1,results_df_team_2))[['time_period','team','_map']]
```


```python
gb = results_df_teams.groupby(['time_period','_map']).team.count()
gb_text = round(gb*100/gb.groupby('time_period').sum(),1).reset_index().rename(columns={'team':'percentage'})
gb_text.percentage = gb_text.percentage.astype(str)+'%'
gb = gb.reset_index()
```


```python
fig = go.Figure()
for _map in maps:
    fig.add_bar(name=_map,x=gb[gb._map==_map].time_period,y=gb[gb._map==_map].team,
                text=gb_text[gb_text._map==_map].percentage,textposition='inside')

fig.update_layout(barmode='stack',legend=dict(traceorder='normal'),yaxis_title='Number of maps played',font=dict(size=10))
fig.show()
```

**Como apontado anteriormente, Nuke é historicamente o mapa menos popular do mapa de jogo. Isso tem mudado recentemente, já que equipes que costumavam eliminar o mapa agora estão banindo mapas como Vertigo.**

**Vertigo, como o mapa mais novo e não convencional, também é o mapa menos popular, provavelmente devido às muitas mudanças que teve em seu curto período competitivo.**

**O período entre Columbus e Cologne 2016 tem a menor quantidade de mapas jogados e também é o mais curto (com menos de 4 meses), enquanto o período entre Boston e Londres 2018 tem a maior quantidade de mapas jogados e também é o mais longo (com mais de 7 meses).**
