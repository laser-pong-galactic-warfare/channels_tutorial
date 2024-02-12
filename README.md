# channels_tutorial

## Informações prévias

### WebSocket frames

No protocolo WebSocket as informações são enviada em sequências de "frames". Eles são pedaços de informação enviados em sequência.

Os frames podem ser de texto (bytes em utf-8), binários (bytes "crus", como arquivos ou imagens), de continuação (para indicar que é uma continuação da anterior) ou de controle (`ping`, `pong` ou `close`).

Cada frame contém os seguintes dados: o tipo de frame, o tamanho da "carga" e a carga em si (a informação que queremos transmitir).

### WSGI vs ASGI

[WSGI](https://wsgi.readthedocs.io/en/latest/what.html) (_Web Server Gateway Interface_) é uma especificação que define como um servidor web deve lidar com aplicações Python de forma síncrona.

[ASGI](https://asgi.readthedocs.io/en/latest/) (_Asynchronous Server Gateway Interface_) é outra especificação. Ela é como o "sucessor espiritual" do WSGI. Como o nome sugere, tem suporte para tarefas assíncronas não bloqueantes e suporta HTTP/2. É útil para comunicação em tempo real, como chat, jogos etc.

**WSGI/ASGI vs CGI**: O uso dessas duas especificações lembra o CGI (_Common Gateway Interface_). Uma das diferenças é que no CGI cada requisição é um processo novo, o que pode acabar não sendo tão eficiente. Já com WSGI/ASGI, a aplicação web em si é carregada somente uma vez e reaproveitada em múltiplas requisições.

**Exemplo de uso**: Digamos que tivessemos uma aplicação web com vários usuários fazendo envios de algum tipo de informação ao mesmo tempo. Com CGI, cada chamada dessa seria um novo processo, consumindo muito recurso do servidor. Com WSGI teríamos um processo apenas, mas que poderia ficar "preso" nessa fila de chamadas, tendo que esperar uma terminar para ir para a próxima. Com ASGI teríamos um processo apenas, mas que não ficaria preso, já que as chamadas poderiam ser realizada em paralelo.

### Async views

A partir da versão 3.5 o Python ganhou a capacidade de realizar tarefas assíncronas. Aproveitando isso, o Django ganhou por padrão uma coisa chamada ["async views"](https://docs.djangoproject.com/en/5.0/topics/async/). São como as views convencionais, mas que permitem execução de código assíncrono (como chamadas a APIs externas) não bloqueante.

**Exemplo de uso**: Digamos que tenhamos uma página que carrega uma lista de filmes vindas de uma API externa. Se ela for síncrona, primeiro precisamos esperar o retorno da API para só depois dar a resposta completa para o cliente. Já se ela for assíncrona, podemos fazer a página ser renderizada e enviada para o cliente e só depois de termos a resposta da API, enviarmos a lista de filmes, sem ter que fazer o usuário ficar esperando tudo.

### Daphne

[Daphne](https://docs.djangoproject.com/en/5.0/howto/deployment/asgi/daphne/) é um servidor ASGI, totalmente em Python, que suporta tanto HTTP quanto WebSocket. Foi feito especificamente para fazer o Channels funcionar, dando suporte à comunicação assíncrona em tempo real. Deve ser inserido no projeto como um app/módulo.

## Conceitos importantes

### Turtles All The Way Down

O Channels se utiliza de uma série de abstrações, feitas a partir de outras abstrações.

A expressão ["turtles all the way down"](https://en.wikipedia.org/wiki/Turtles_all_the_way_down) remete a ideia de uma [regressão infinita](https://pt.wikipedia.org/wiki/Regress%C3%A3o_infinita). No caso do Channels essa seria uma regressão de abstrações.


### [Escopos e eventos](https://channels.readthedocs.io/en/latest/introduction.html#scopes-and-events)

As conexões feitas em no Channels com ASGI se dividem em dois componentes: um escopo e uma série de eventos.

**Escopo**: É o conjunto de detalhes de uma conexão, como origem da requisição, IP etc. Para requisiões HTTP ele existe para uma requisição única, enquanto para WebSockets ele dura enquanto a conexão estiver ativa.

**Eventos**: Enquanto um escopo existir uma série de eventos pode ocorrer. Eventos são "interações" feitas pelo cliente, como fazer uma requisição HTTP ou enviar um WebSocket frame. Uma aplicação Channels/ASGI vai ser instanciada uma vez por escopo, e nessa instância serão passados os eventos para que a aplicação decida o que fazer.

### Consumers

Consumers são a principal abstração do Channels. Eles tem esse nome por "consumirem eventos". Eles permitem a criação de aplicações ASGI de forma simplificada.

Eles são como as views do Django. A diferença é que as Views foram feitas para executar uma função e encerrar suas atividades depoius disso. Enquanto os consumers foram feitos para ficar rodando até que não sejam mais necessários, ou seja, quando um escopo for encerrado.

Eles nos permitem criar funções que serão chamadas sempre que um evento ocorrer, sem a necessidade de criar loops para isso. Além disso permitem escrever código tanto síncrono quanto assíncrono, além de cuidar automaticamente de multithreading (se/quando necessário).

### Resumo até aqui

Daphne (ou outro servidor ASGI) vai receber as conexões e requisições e iniciar um processo da nossa aplicação e mantém esse processo rodando. Enquanto isso, Daphne vai repassar as requisições/eventos/frames para os consumers da nossa aplicação de forma assíncrona.
