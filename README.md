# - 🔒 Projeto de Bolsas DevSecOps/AWS,  Compass UOL, abril 2025 🔒 -

## ✅ Servidor Nginx com checagem em tempo real por Webhook ✅

## 📜 0 - Breve resumo >
        O projeto consiste em levantar, em algum ambiente Linux (seja por meio de máquinas virtuais, WSL ou Boot nativo), uma página HTML simples com servidor Nginx, onde este é monitorado por um Script.  
    
        Este Script é executado a cada 1 minuto, e caso o servidor esteja fora do ar, retorna uma mensagem por meio de Webhook relatando o erro. Além disso, são armazenadas todas as informações coletadas pelo Script em um Log.
---
## 🐧 1 - Ambiente Linux e Discord >
        Utilizei em meu projeto uma Máquina Virtual, por meio do aplicativo Oracle VirtualBox. Criei a máquina com os requisitos mínimos para a utilização do Debian Headless, distro que possuo maior familiaridade. Além disso, habilitei a placa de rede em modo Bridge, para evitar possíveis erros de conexão.
![Primeiro print](/Prints/1.1%20-%201.png)

       Depois disso, criei um servidor no Discord e fiz um Webhook personalizado no chat principal.
![Segundo print](/Prints/1.1%20-%202.png)

        Então, já dentro da VM, fiz a instalação dos pacotes que eu precisaria para a realização do projeto:
>`apt-get install nginx` - Para conseguir levantar o servidor da página web;  
>`apt-get install samba` - Para conseguir compartilhar arquivos da minha máquina windows com a VM;  
>`apt-get install curl` - Para verificar a conectividade da URL;
---
## 👾 2 - Preparando o Sistema >
        Após a instalação dos pacotes, eu comecei a preparar a landing page do site. O primeiro passo foi averiguar se o Nginx estava de pé.
![Terceiro print](/Prints/1.2%20-%201.png)

        Como nenhum erro aconteceu, fui até meu Visual Studio Code para criar o HTML e o CSS.
        Ao finalizar, utilizei o Samba para passar o arquivo da minha máquina principal para a VM:
>`cd /` - Para ir até o diretório Root;  
>`mkdir dados` - Para criar a página que o Samba utilizará para comunicar as máquinas;  
>`chmod 777 dados` - Para dar permissões gerais de escrita, leitura e execução da pasta;  
>`vi /etc/samba/smb.conf` - e então configurei conforme a imagem:

![Quarto print](/Prints/1.2%20-%202.png)

        Após isso, bastou passar o arquivo por meio da pasta dados:
![Quinto print](/Prints/1.2%20-%203.png)
![Sexto print](/Prints/1.2%20-%204.png)

        E por fim:
>`mv oi.txt //var/var/www/html` - Para então levar o arquivo html até a pasta onde o Nginx lê seus arquivos para o site;  
>`mv index.nginx-debian.html index.nginx-debian.html.original` - Para salvar o arquivo padrão para emergências;  
>`vi index.nginx-debian.html` - Para criar um novo arquivo base que o Nginx vai ler;  

![Sétimo print](/Prints/1.2%20-%205.png)

>`:r Oi.txt` - Dentro do `index.nginx-debian.html` para colar todo o conteúdo de Oi.txt lá;

![Oitavo print](/Prints/1.2%20-%206.png)

        E então, quando botamos o IP no navegador para testar se o Nginx localizou e utilizou o novo arquivo index:
![Décimo print](/Prints/1.2%20-%207.png)

---
## #️⃣ 3 - Criando o Script >
>`cd ` - Para ir para Home;  
>`vi .bashrc` - Para editar o .bashrc;  

![Décimo primeiro print](/Prints/2.1%20-%201.jpg)

        A última linha serve para criar uma variável de ambiente. Na parte censurada está o link do Webhook que criei mais cedo.

        Em seguida, utilizei o comando Curl para testar a funcionalidade da variável:
>`curl -H "Content-Type: application/json" -d '{"content": "Servidor de Pé"}' $BotNotf` - Cria uma mensagem em Json, com a mensagem especificada após `-d` para o link da nossa variável `$BotNotf`;

![Décimo segundo print](/Prints/2.1%20-%202.png)

        Como foi um sucesso, criei um novo diretório em Home, chamado "scripts", para armazená-lo. Dentro dele, criei um arquivo "monitoramento.sh" e comecei a criar o Script.
![Décimo terceiro print](/Prints/2.1%20-%203.png)
>`#!/bin/bash` - Serve para indicar o interpretador, nesse caso: Bash;  
>`URL="http://192.168.0.22"` - Cria uma variável com o endereço do servidor Nginx;  
>`STATUS_HTTP=$(curl -s -o /dev/null -w "%{http_code}" $URL)` - O curl faz, silenciosamente e descartando o corpo no diretório `/dev/null`, a requisição do código http do servidor através da variável `URL`, e o escreve na variável `STATUS_HTTP`;  
>`if -> fi` - A condição exige que o código http seja igual (`-eq`) a 200, e se for, ele executa o mesmo Curl da etapa anterior;  

        Com o script salvo, dei a permissão de execução para todos do sistema, e então o testei:
![Décimo quarto print](/Prints/2.1%20-%204.jpg)

        E então chegou a hora de "agendar" sua execução para a cada 1 minuto. Pra isso, utilizei o crontab.
>`crontab -e` - para acessar o crontab;

![Décimo quinto print](/Prints/2.1%20-%205.png)
>`* * * * * /bin/bash /root/monitoramento.sh` - Os asteriscos indicam que vai ser repetido a todo minuto, e o resto manda executar o script que criei com o interpretador bash;

        Precisei também indicar pro script onde estava a variável de ambiente que criei, pois o crontab não conseguiria interpretá-lo se não fizesse.

>`source ~/.bashrc` - Bastou colocar isso após o shebang;

        Após isso, a cada minuto, automaticamente, o script era executado:
![Décimo sexto print](/Prints/2.1%20-%206.png)
        
        Mas o objetivo não é notificar quando o servidor estivesse em pé, por isso, comentei todo o if feito anteriormente, e criei um que verificava se o servidor não fosse encontrado.
![Décimo sétimo print](/Prints/2.1%20-%207.png)
>`if -> fi` - As diferenças entre o código que verifica quando o servidor está online são: agora o script verifica quando o código http **NÃO** é igual (-ne), e então realiza o curl, que agora mostra o `$STATUS_HTTP` em sua mensagem;

        E então, alterei o endereço ip armazenado na variável URL para fazer o teste. Sucesso:
![Décimo oitavo print](/Prints/2.1%20-%208.png)

---
## 📝 4 - Armazenando os resultados em log >
        Para criar o log, bastou alterar um pouco o código do script:
![Décimo nono print](/Prints/3.1%20-%201.png)
>`ARQUIVO_LOG` - Armazena na variável o caminho até o arquivo log que será utilizado pelo script;  
>`echo "$(date '+%Y-%m-%d %H:M:%S') - Status: $STATUS_HTTP" >> $ARQUIVO_LOG` - Faz com que uma mensagem indicando a data e hora exatas, junto do código http seja escrita na última linha do arquivo log toda vez que o script for executado;  
>`echo "$(date '+%Y-%m-%d %H:M:%S') - REGISTRO: ********" >> $ARQUIVO_LOG` - Faz outra mensagem ser escrita na última linha, escrevendo o status do servidor. Fiz duas variações, a primeira para alertar que o servidor estava em pé e outra indicar que estava fora do ar;

        Como o objetivo do desafio era indicar quando o servidor estivesse fora do ar, comentei a parte do código que indicava o servidor online após testar a funcionalidade.
>`cat //var/log/monitoramento.log` - Para podermos analisar o arquivo do log:

![Vigésimo print](/Prints/3.1%20-%202.png)

---
## 🧠 5 - Conclusão >
        Por fim, consegui alcançar as metas do desafio e evoluir meus conhecimentos sobre Linux e Scripts com a jornada, alcançando satisfação com o resultado final.
---