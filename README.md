# BP

Lightweight server and client.

## TODOs
* V0 protocolo
* ~criar server (hello world)~
* ~Criar client (hello world)~
* Utilizar uma versão asyncrona do socket
* Adicionar E2EE na mensagem
* Implementar connect(?) do protocolo
* Registro + autenticacao?
* Criar interface da tui
* Adicionar permissão de envio de mensagem (solicitar, rejeitar, aceitar)
* Adicionar envio de DM
* Impor TTL da conexão
* Adicionar suporte a grupo
    * Acho que no documento de especificação vale a pena definir o que é grupo e o que é canal antes dessa implementação
* Criar canais
* Mensagem por canal

## mvp 
* e2e encryption
* ter suporte a channel
* mensagens privadas
* suporte a TLS
* apenas texto por enquanto
* stateless server.

### server (zig)
### client tui (go)

### Dúvidas

1. Quão viável é termos suporte a channel sem um stateful server?
2. Quão viável é termos autenticação sem um stateful server?

As dúvidas acima me fazem pensar que precisaremos de um stateful server parcial, ao menos.

______

## nice to have
* stateful server
* clients com suporte a wyswig
* clients com suporte a markdown
* Suporte à chamada de voz
* Suporte à chamada de vídeo
* p2p
* busca global (conteudo, contatos, feature, man, help)
* times ??
