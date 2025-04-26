# Projeto Linux - PB Compass UOL
## Monitoramento de um servidor web com Nginx<br/>

### Etapas:
* Configurar um ambiente Linux;
* Subir um servidor Nginx;
* Criar uma página html simples que será exibida dentro do servidor;
* Criar um script que verifiquea cada 1 minuto se o site está disponível;
* Caso o site esteja indisponível, o script deve enviar uma notificação informando a indisponibilidade do site via Webhook para algum canal (Discord, Telegram ou Slack);
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


