# Projeto Linux - PB Compass UOL
## Monitoramento de um servidor web com Nginx<br/>

### Etapas:
* Configurar um ambiente Linux;
* Subir um servidor Nginx;
* Criar uma página html simples que será exibida dentro do servidor;
* Criar um script que verifique a cada 1 minuto se o site está disponível. Caso o site esteja indisponível, o script deverá enviar uma notificação informando a indisponibilidade do site via Webhook para algum canal (Discord, Telegram ou Slack);
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

![image](https://github.com/user-attachments/assets/873a07eb-332b-43e5-9306-80641f1087e8)

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
  * Em casos de código diferente de 200, imprime o momento de verificação e o resultado da verificação em um arquivo de log, e envia um alerta a um dado servidor de Discord informando que o site está fora do ar.
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
### Automatização do script:
Com o uso do `cron` para agendamento de serviços, usei o comando `sudo crontab -e` para acessar e editar o arquivo crontab, que contem os agendamentos. Nesse arquivo adicionei a seguinte linha:
```
* * * * * /var/www/html/monitoringscript.sh
```
Onde os asteriscos, da esquerda para a direita, respectivamente, significam minuto, hora, dia do mês, mês e dia da semana; seguido do arquivo que será executado.
> [!NOTE]
> Nesse caso pode ser tanto `* * * * *` que significa "todo minuto" quanto `*/1 * * * *` que significa "a cada 1 minuto".

## Quinta etapa - criação do arquivo de log:
Dentro do diretório `/var/log` usei o comando `touch` para criar o arquivo `monitoringscript.log` que é apontado na linha `echo "Verificação executada em $(date) - Código HTTP: $STATUS" >> /var/log/monitoringscript.log` dentro do arquivo `monitoringscript.sh`.<br/>

Para visualizarmos o conteúdo do arquivo de log, utilizei o comando `cd /var/log` para entrar no diretório onde o arquivo está localizado e em seguida o comando `cat monitoringscript.log` para visualizar o conteúdo a seguir:

![image](https://github.com/user-attachments/assets/1197650b-c10c-4d36-b271-84bb09e46ebe)
> [!NOTE]
> As linhas com código HTTP 000 são de momentos em que simulei uma queda do servidor

## Testando:
Para fins de teste utilizei o comando `sudo systemctl stop nginx` para parar o servidor e simular uma queda, resultado no seguinte alerta em meu Discord:<br/>
![image](https://github.com/user-attachments/assets/c4acf6f1-6471-438d-977b-3b332d8d635a)

Para que o servidor voltasse ao ar, utilizei o comando `sudo systemctl start nginx`.
