# GR1 robot

Queremos treinar uma politica que se adapte bem ao robo real.

Este e basicamente o problema do Sim2Real, e um problema extremamente dificil, pois e muito complicado pois utilizando servo motores baratos. Primeiro porque este motores tem uma precisao moderada alem de nao serem potentese, com isso replicar os movimentos aprendidos pela politica se torna uma missao complexa.

## Projetar o modelo 3D preciso do robo

O primeiro passo que devemos realizar consiste em construir um modelo CAD 3D preciso que **aproxime** bem em relacao a estrutura fisica e as caracteristicas fisicas como massa que nosso robo tera.Um aspecto necessario e que este modelo 3D, deve ser exportado em formatos suportados em formatos conhecidos ou *standard* na industria tais como: **URDF,MJCF ou USD**.

Sendo assim, para realizar o projeto do prototipo utilizamos o Fusion360, no qual se mostrou uma ferramenta bastante util e precisa pois foi possivel especificar o material de cada peca do robo.

Com o modelo projetado, realizamos a construcao do protitpo utilizando impressao 3D, mas preciso praticamente todas as pecas do robo foram impressas utilizando o filamento **PLA+** da empresa **eSUN** na impresora **Creality K1C**, exceto por algumas partes adaptcoes feitas que foram impressas utilizando material **TPU** da marca **eSUN** na impressora **Bambu Lab X1C**.Todas as pecas, utilizaram um preenchimento (infill) de 15%.

Como comentamos, estamos diante de um problema de *sim2Real* e pra isso precisamos ter o modelo 3D o mais proximo do prototipo do mundo real para que a politica consiga aprender comportamentos locomotores ultraprecisos que se aproximem dos padroes locomotores de um bipede sob acao, da fisica do planeta terra. 

Portanto, na tentativa de ter o modelo 3D mais preciso possivel utilizamos a estrategia de coletar as informacoes bastante precisas de massa de filamento PLA gasto para produzir as pecas em 3D, fornecidas pelo software fatidor OrcaSlicer e com isso, foi possivel construir um modelo 3D que sera utilizado na simulacao bastante proximo do prototipo real.


## Importando o modelo 3D do robo para o simulador

Agora com o software pronto utilizamos o plug-in chamado fusion2urdf, para exportar a montagem do robo projetado no software CAD Fusion360 direto para o nosso simulador.

Este plug-in exporta as descricoes da estrutura da malha do robo projetada no software 3D para o formato URDF/MJCF para um formato MJX leve e preciso que o simulador consegue ler e renderizar. E portanto, com isso temos uma descricao MJCF que descreve com precisao as massas e os momentos de inercia do robo completo.



## Motores e servoatuadores
Um aspecto de extrema importancia, para se ter um modelo preciso pricipalmente no contexto do sim2Real e modelar o comportamento dos motores/servoatuadores. Para isso, utilizamos uma ferramenta que nos auxiliou neste processo de modelagem chamada BAM (Better Actuator Model) esta ferramenta nos ajuda no seguinte contexto: 

>Modelos precisos de servoatuadores são essenciais para a simulação de sistemas robóticos. Isso é particularmente importante ao realizar Aprendizado por Reforço (AR) em robôs reais, pois a precisão do modelo impacta diretamente a transferibilidade da política aprendida.

> Os simuladores atuais amplamente utilizados, como MuJoCo ou IsaacGym geralmente modelam o atrito atraves da implementacao da modelagem do atrito de Coulumb-Viscoso, que e muito simplista para representar com precisao os fenomenos de atrito complexos que o robo vai experimentar em operacao nesse caso seria interessante ter modelos como o efeito Stribeck, a dependencia da carga ou os efeitos quadraticos.

Portanto, a ferramenta BAM propoe:

- Criar um processo de identificação para ajustar modelos de fricção a partir de trajetórias registradas;
- Fornecer um conjunto de modelos de fricção estendidos que capturam fenômenos de fricção complexos,
- Compartilhar uma biblioteca de modelos de atrito identificados para servos comuns como o utilizado pro nos Feetech STS3215;
- Fornecer uma API simples para utilizar esses modelos de atrito em simuladores como o MuJoCo.

Para mais detalhes: (leia este [artigo](https://arxiv.org/pdf/2410.08650v1))

É crucial para nosso robo que o simulador simule os motores com precisão, pois treinaremos uma política (uma rede neural) para gerar posições dos motores com base em entradas sensoriais (posições/velocidades dos motores, IMU e sensores de fim de curso presente nos pes do robo). Se os motores se comportarem de maneira diferente na simulação do que no mundo real, a política não funcionará ou, na pior das hipóteses, produzirá padroes locomotores altamente nao lineares e movimentos caóticos .

E com a ferramenta BAM, podemos exportar os principais parametros necessarios para unidades MuJoCo que estara presente no nosso arquivo de descricao MJFC.Como estamos utilizando motores Feetech STS3215 encontraremos no nosso arquivo de descicao MJFC o seguinte:

```json
    "kt": 1.21164135295077,
    "R": 2.6761663274455603,
    "armature": 0.02840336348682085,
    "q_offset": -0.05116067663549731,
    "friction_base": 0.05239296084748866,
    "friction_viscous": 0.05908515565091076,
    "model": "m1",
    "actuator": "sts3215"

```
Alem deste, valores estarao presentes nas propriedades dos atuadores e das juntas tambem:

- amortecimento;
- perda por atrito;
- kp;
- alcance de forca;

## Treinando uma nova Politica

Usamos nossa própria estrutura baseada no [mujoco playground](https://github.com/google-deepmind/mujoco_playground), o [Open Duck Playground](https://github.com/apirrone/Open_Duck_Playground)

No ambiente [joystick](https://github.com/apirrone/Open_Duck_Playground/blob/main/playground/open_duck_mini_v2/joystick.py), você pode tentar ativar/desativar diferentes recompensas, escrever as suas próprias, brincar com os pesos, ruído, aleatorização etc.

Obtivemos bons resultados implementando a recompensa por imitação descrita pela Disney em seu [artigo BDX](https://github.com/apirrone/Open_Duck_Playground/blob/main/playground/open_duck_mini_v2/joystick.py).

Para usar essa recompensa, precisamos de movimentos de referência. Criamos [este repositório](https://github.com/apirrone/Open_Duck_reference_motion_generator) para gerar esses movimentos usando um mecanismo de caminhada paramétrica. Seguindo as instruções lá, você pode gerar um arquivo `polynomial_coefficients.pkl` que contém os movimentos de referência. Já existe um arquivo desse tipo no repositório do playground, no diretório `data/`.

Após o treinamento da sua política, você pode tentar executá-la no robô real usando [este script](https://github.com/apirrone/Open_Duck_Mini_Runtime/blob/v2/scripts/v2_rl_walk_mujoco.py) no repositório do runtime. Certifique-se de ter concluído todas as etapas da [lista de verificação](https://github.com/apirrone/Open_Duck_Mini_Runtime/blob/v2/checklist.md) antes de executar o script.

## Treinamento da politica
Seguir este passos:
https://github.com/apirrone/Open_Duck_Playground

## Gerando movimentos para treinar uma imitational reward

Seguir estes passos:
https://github.com/apirrone/Open_Duck_reference_motion_generator

Biblioteca para gerar os movimentos: https://github.com/Rhoban/placo

Ambiente alternativo utilizando Isaac Gym: https://github.com/rimim/AWD

Dar uma olhada neste outra paret do repo:

https://github.com/SteveNguyen/openduckminiv2_playground


## Levando o modelo treinado para o robo

Olhar: https://github.com/apirrone/Open_Duck_Mini_Runtime

---

## Notas:
O que sao os formatos URDF,MJCF e Open USD?

Estes formatos sao formatos suportados por alguns do simuladores de fisica realistica mais precisos e utilizados da industria

- URDF: Unified Robot Description File, basicamente consiste em um conjunto de tags xml, que representam a estrutura ou esqueleto do robo, este formato e muito utilizado junto com o ROS2 e seu arquivo de visualizacao de malhas geometricas Rviz.

- MJCF: TODO


- USD: O Universal Scene Description, basicamente assim como o URDF, tambem e 


## Referencias
Parametros dos modelos de atrito de servoatuadores comerciais:

- https://github.com/Rhoban/bam/tree/main/bam/params

- https://github.com/syuntoku14/fusion2urdf

- https://developer.nvidia.com/blog/using-openusd-for-modular-and-scalable-robotic-simulation-and-development/

- https://github.com/apirrone/Open_Duck_Playground/blob/main/playground/open_duck_mini_v2/joystick.py