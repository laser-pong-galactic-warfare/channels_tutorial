# channels_tutorial

## Informações prévias

### WSGI vs ASGI

[WSGI](https://wsgi.readthedocs.io/en/latest/what.html) (_Web Server Gateway Interface_) é uma especificação que define como um servidor web deve lidar com aplicações Python de forma síncrona.

[ASGI](https://asgi.readthedocs.io/en/latest/) (_Asynchronous Server Gateway Interface_) é outra especificação. Ela é como o "sucessor espiritual" do WSGI. Como o nome sugere, tem suporte para tarefas assíncronas não bloqueantes e suporta HTTP/2. É útil para comunicação em tempo real, como chat, jogos etc.

**WSGI/ASGI vs CGI**: O uso dessas duas especificações lembra o CGI (_Common Gateway Interface_). Uma das diferenças é que no CGI cada requisição é um processo novo, o que pode acabar não sendo tão eficiente. Já com WSGI/ASGI, a aplicação web em si é carregada somente uma vez e reaproveitada em múltiplas requisições.

**Exemplo de uso**: Digamos que tivessemos uma aplicação web com vários usuários fazendo envios de algum tipo de informação ao mesmo tempo. Com CGI, cada chamada dessa seria um novo processo, consumindo muito recurso do servidor. Com WSGI teríamos um processo apenas, mas que poderia ficar "preso" nessa fila de chamadas, tendo que esperar uma terminar para ir para a próxima. Com ASGI teríamos um processo apenas, mas que não ficaria preso, já que as chamadas poderiam ser realizada em paralelo.

### Async views

A partir da versão 3.5, o Python ganhou a capacidade de realizar tarefas assíncronas. Aproveitando isso, o Django ganhou por padrão uma coisa chamada ["async views"](https://docs.djangoproject.com/en/5.0/topics/async/). São como as views convencionais, mas que permitem execução de código assíncrono (como chamadas a APIs externas) não bloqueante.

**Exemplo de uso**: Digamos que tenhamos uma página que carrega uma lista de filmes vindas de uma API externa. Se ela for síncrona, primeiro precisamos esperar o retorno da API para só depois dar a resposta completa para o cliente. Já se ela for assíncrona, podemos fazer a página ser renderizada e enviada para o cliente e só depois de termos a resposta da API, enviarmos a lista de filmes, sem ter que fazer o usuário ficar esperando tudo.

### Daphne
[Daphne](https://docs.djangoproject.com/en/5.0/howto/deployment/asgi/daphne/) é um servidor ASGI, totalmente em Python, que suporta tanto HTTP quanto WebSocket. Foi feito especificamente para fazer o Channels funcionar, dando suporte à comunicação assíncrona em tempo real Deve ser inserido no projeto como um app/módulo.