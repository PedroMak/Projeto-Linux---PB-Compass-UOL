# Projeto Linux - PB Compass UOL
## Monitoramento de um servidor web com Nginx<br/>

### Etapas:
* Configurar um ambiente Linux;
* Subir um servidor Nginx;
* Criar uma página html simples que será exibida dentro do servidor;
* Criar um script que verifiquea cada 1 minuto se o site está disponível. Caso o site esteja indisponível, o script deve enviar uma notificação informando a indisponibilidade do site via Webhook para algum canal (Discord, Telegram ou Slack);
* O script deve armazenar os logs da execução em um local no servidor.

## Primeira etapa - configuração do ambiente:
Primeiro rodei o seguinte comando no PowerShell:
```
wsl --install
```
Em seguida, instalei o seguinte subsystem WSL2 da Microsoft Store:<br/>
![image](https://github.com/user-attachments/assets/ae1bbd12-2b36-4598-aae7-76d61429a354)

## Segunda etapa - subir servidor Nginx:
Primeiro fiz uma atualização dos meus pacotes com os comandos:
```
sudo apt update
sudo apt upgrade
```
Em seguida fiz a instalação do nginx com o seguinte comando:
```
sudo apt install nginx
```
Após a instalação podemos rodar o comando `systemctl status nginx` para verificar o status do servidor que, pela imagem ![image](https://github.com/user-attachments/assets/141fe4b9-5df0-4b75-9215-8ebbb5f5c366) vemos que se encontra ativo.

## Terceira etapa - criação da página html:
Dentro do diretório `/var/www/html` encontra-se um arquivo chamado `index.nginx-debian.html` com uma apresentação simples sobre o Nginx. Mative a estrutura do site e apenas alterei o texto.<br/>
![image](https://github.com/user-attachments/assets/45d5e5e0-3093-4e73-a07f-6d60233ef0ab)

## Quarta etapa - criação do script de verificação:
Ainda dentro do diretório `/var/www/html` usei o comando `touch` para criar um arquivo chamado `monitoringscript.sh`, onde escrevi o código a seguir:
```
#!/bin/sh

STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://ip.ip.ip.ip/)

if [ "$STATUS" = "200" ]; then
        echo "Verificação executada em $(date) - Código HTTP: $STATUS" >> /var/log/monitoringscript.log

else
        echo "Verificação executada em $(date) - Código HTTP: $STATUS" >> /var/log/monitoringscript.log
        curl -H "Content-Type: application/json" \
             -X POST \
             -d "{\"content\": \"Website DOWN.\"}" \
             "link para webhook aqui"

fi
```
### Explicando o código:
* `#!/bin/sh` - indica ao sistema operacional que o script deve ser executado usando o shell padrão;
  
* `STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://ip.ip.ip.ip/)` - armazena na variável `STATUS` o código HTTP;
  * `curl` - comando utilizado para transferir dados entre um cliente e um servidor utilizando, no caso, o protocolo HTTP;
  * `-s` - oculta a barra de prograsso ou mensagens;
  * `-o /dev/null` - descarta o conteúdo HTML do site;
  * `-w "%{http_code}` - exibe apenas o código HTTP retornado;

* `if [ "$STATUS" = "200" ]; then` - verifica o código HTTP recebido;
  * Em caso de código 200, executa a seguinte linha, que imprime o momento de verificação e o resultado da verificação em um arquivo de log.
     ```
     echo "Verificação executada em $(date) - Código HTTP: $STATUS" >> /var/log/monitoringscript.log
     ```
  * Em casos de código diferente de 200, imprime o momento de verificação e o resultado da verificação em um arquivo de log e envia um alerta a um dado servidor de Discord informado que o site está fora do ar.
    ```
    else
        echo "Verificação executada em $(date) - Código HTTP: $STATUS" >> /var/log/monitoringscript.log
        curl -H "Content-Type: application/json" \
             -X POST \
             -d "{\"content\": \"Website DOWN.\"}" \
             "link para webhook aqui"
    ```
    * `-H "Content-Type: application/json"` - informa que o corpo da mensagem está em JSON;
    * `-X POST` - definição de método HTTP a ser utilizado;
    * `-d "{\"content\": \"Website DOWN.\"}"` - corpo da mensagem a ser enviada;
    * E por último o link do Webhook.
     

