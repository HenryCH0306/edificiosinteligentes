# edificiosinteligentes
Esse é um projeto feito a partir de um problema proposto (ODS9), que visa modernizar edifícios, essa é a versão feita por mim, mas caso haja interesse, é possível modificar/ melhorar o código para que se adeque as condições do usuário.

<h2>IDEIA:</h2>

O objetivo inicial era criar um sistema IoT que permitisse um edifício pudesse manter-se em um estado ideal de forma autônoma, onde a partir de sensores e atuadores seria possível monitorar e controlar não só a temperatura, mas também a pureza do ar do ambiente, e configurar para que se ajustem automaticamente para o estado que considera "ideal". 

<h2>Materiais/Funcionamentos</h2> 
ESP32 (Micro controlador com módulo de Wi-fi e bluetoot embutido, responsável pela comunicação entre os sensores e a interface de controle do sistema).
Módulo de sensor IR (Sensor infravermelho responsável por ligar e desligar os aparelhos funcionais)
Sensor DHT - 22 (Sensor de temperatura e humidade)
Ar-condicionado Fujitsu (Aparelho funcional capaz de regular a temperatura do ambiente, caso use outro modelo de ar-condicionado, deverá mudar o código para o modelo do ar-condicionado usado).

Para esse projeto foi utilizado o software Arduino IDE para configuração do ESP32, utilizando o código do arquivo "Edificiosinteligentes" (Obs: Caso haja erro na hora da transferência do código, tente segurar o boot na hora da transferência para o ESP32.). 

<h2>Conexões:</h2>

Módulo IR </br>
VCC: 3V3</br>
GND: GND</br>
OUT: D23</br>

Sensor DHT - 22</br>
VCC: 3V3</br>
GND: GND</br>
DATA: D21</br>
(Obs: É extremamente recomendado o uso de uma protoboard para realização do projeto.)

<h2>Interface de comunicação</h2>
Utilizei o Protocol MQTT Box para realizar o projeto, porem é possível realiza-lo de outras formas, como utilizando um Broker público como o do mostrado no código.

PS: Essa é uma versão em pequena escala do projeto, o projeto em si conta com diversos sensores e atuadores para ter o maior controle possível de um edifício.
