#!/bin/bash

BLUE='0;35'
NC='\033[0m'
VERSAO=11

echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) OlÃ¡, serei seu assistente para instalaÃ§Ã£o."
echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Verificando se vocÃª possui o Java instalado..."
sleep 2

java -version
if [ $? -eq 0 ]; then
    echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Voce ja¡ tem o Java instalado!"
else
    echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Opa! Nao identifiquei nenhuma versao do Java instalada, mas sem problemas, irei resolver isso agora!"
    echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Confirme para mim se realmente deseja instalar o Java (S/N)?"
    read -r inst
    if [ "$inst" == "S" ]; then
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Ok! Voce escolheu instalar o Java ;D"
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Adicionando o repositorio!"
        sleep 2
        sudo add-apt-repository ppa:webupd8team/java -y
        clear
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Atualizando! Quase la..."
        sleep 2
        sudo apt update -y
        clear

        if [ "$VERSAO" -eq 11 ]; then
            echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Preparando para instalar a versÃ£o 17 do Java. Confirme a instalaÃ§Ã£o quando solicitado ;D"
            sudo apt install openjdk-17-jdk openjdk-17-jre -y
            clear
            echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Java instalado com sucesso!"
            echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Agora irei atualizar os pacotes do seu sistema operacional!"
            sudo apt update && sudo apt upgrade -y
        fi
    else
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Voce optou por nao instalar o Java por enquanto, ate a proxima entao!"
    fi
fi

echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Irei verificar se vocÃª possui o Docker em sua mÃ¡quina..."
docker --version >/dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Voce ja tem o Docker instalado!"
else
    echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Opa! Nao identifiquei nenhuma versao do Docker instalada, mas sem problemas, irei resolver isso agora!"
    echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Podemos instalar o Docker em sua maquina (S/N)?"
    read -r inst
    if [ "$inst" == "S" ]; then
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Ok! Voce escolheu instalar o Docker ;D"
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Instalando o Docker em sua maquina!"
        sleep 2
        sudo apt install docker.io -y
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Ativando o Docker na maquina"
        sleep 28
        sudo systemctl start docker
        sleep 28
	echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Habilitando o servico"
        sudo systemctl enable docker
        sleep 28
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Agora iremos criar um container no Docker para ter um backup dos dados!"
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Fazendo o download da imagem do banco de dados"
        sudo docker pull mysql:5.7
        sleep 20
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Verificando imagem"
        sudo docker images
        clear
        sleep 5
    fi
fi
if [ "$(sudo docker ps -aqf name=Stabillis)" ]; 
	then
        echo -e "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Voce ja possui o banco de dados em seu sistema! Vamos inicializar o container"
        sleep 5
        sudo docker start Stabillis >/dev/null 2>&1
        sleep 5
        echo -e "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Container Stabillis inicializado!"
        sudo docker ps -a
    else
        echo -e "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Não encontramos nenhum banco do MySQL em sua maquina, vamos resolver isso!"
        sleep 5
        echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Agora iremos criar e configurar a aplicacao do banco de dados dentro do container"
        sudo docker run -d -p 3306:3306 --name Stabillis -e "MYSQL_DATABASE=stabillis" -e "MYSQL_ROOT_PASSWORD=urubu100" mysql:5.7
        sleep 20
        clear
        sudo docker ps -a
        sudo docker exec -i Stabillis mysql -uroot -purubu100 -e "USE stabillis; CREATE TABLE IF NOT EXISTS Empresa (idEmpresa INT NOT NULL AUTO_INCREMENT, NomeEmpresa VARCHAR(45) NULL, CNPJ CHAR(14) NULL, TelefoneFixo CHAR(10) NULL, CEP CHAR(8) NULL, Logradouro VARCHAR(45) NULL, Complemento VARCHAR(45) NULL, Bairro VARCHAR(45) NULL, Cidade VARCHAR(45) NULL, Estado VARCHAR(45) NULL, PRIMARY KEY (idEmpresa));CREATE TABLE IF NOT EXISTS Perfil (idPerfil INT NOT NULL AUTO_INCREMENT, TipoPerfil VARCHAR(45) NULL, PRIMARY KEY (idPerfil));CREATE TABLE IF NOT EXISTS Usuario (idUsuario INT NOT NULL AUTO_INCREMENT, NomeUsuario VARCHAR(45) NULL, TelefoneCelular CHAR(11) NULL, Email VARCHAR(100) NULL, Senha VARCHAR(45) NULL, FK_Perfil INT NOT NULL, FK_Empresa INT NOT NULL, PRIMARY KEY (idUsuario), INDEX fk_CadastroColaborador_CadastrarEmpresa_idx (FK_Empresa), INDEX fk_Usuario_Cargo1_idx (FK_Perfil), CONSTRAINT fk_CadastroColaborador_CadastrarEmpresa FOREIGN KEY (FK_Empresa) REFERENCES Empresa (idEmpresa), CONSTRAINT fk_Usuario_Cargo1 FOREIGN KEY (FK_Perfil) REFERENCES Perfil (idPerfil));CREATE TABLE IF NOT EXISTS Status (idStatus INT NOT NULL AUTO_INCREMENT, TipoStatus VARCHAR(45) NULL, PRIMARY KEY (idStatus));CREATE TABLE IF NOT EXISTS Maquina (idMaquina INT NOT NULL AUTO_INCREMENT, nomeMaquina VARCHAR(45) NULL, FK_Status INT NOT NULL, capacidadeMaxRAM DECIMAL(10,2) NULL, capacidadeMaxDisco DECIMAL(10,2) NULL, capacidadeMaxCPU DECIMAL(5,2) NULL, Arquitetura CHAR(7) NULL, SistemaOperacional VARCHAR(45) NULL, FK_Empresa INT NOT NULL, PRIMARY KEY (idMaquina), INDEX fk_Maquina_Empresa1_idx (FK_Empresa), INDEX fk_Maquina_Status1_idx (FK_Status), CONSTRAINT fk_Maquina_Empresa1 FOREIGN KEY (FK_Empresa) REFERENCES Empresa (idEmpresa), CONSTRAINT fk_Maquina_Status1 FOREIGN KEY (FK_Status) REFERENCES Status (idStatus));CREATE TABLE IF NOT EXISTS Captura (idCaptura INT NOT NULL AUTO_INCREMENT, usoRAM DECIMAL(10,2) NULL, usoCPU DECIMAL(10,2) NULL, usoDisco DECIMAL(10,2) NULL, bytesRecebidos VARCHAR(45) NULL, bytesEnviados VARCHAR(45) NULL, tempoAtividade VARCHAR(100), dataHora DATETIME NULL, FK_Maquina INT NOT NULL, PRIMARY KEY (idCaptura), INDEX fk_Captura_Maquina1_idx (FK_Maquina), CONSTRAINT fk_Captura_Maquina1 FOREIGN KEY (FK_Maquina) REFERENCES Maquina (idMaquina)); CREATE TABLE IF NOT EXISTS Limite (idLimite INT NOT NULL, FK_Empresa INT NOT NULL, maximoCPU DECIMAL(10,2) NULL, maximoRAM DECIMAL(10,2) NULL, maximoDisco DECIMAL(10,2) NULL, PRIMARY KEY (idLimite), INDEX fk_Limite_Empresa1_idx (FK_Empresa), CONSTRAINT fk_Limite_Empresa1 FOREIGN KEY (FK_Empresa) REFERENCES Empresa (idEmpresa)); INSERT INTO Status Values (1, 'Ativo'), (2, 'Inativo');"
        echo -e "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Tabelas criadas com sucesso!"
    fi

echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Todos os requisitos foram instalados com sucesso!"


echo "$(tput setaf 10)[Assistente - Stabillis]:$(tput setaf 7) Inciando AplicaÃ§Ã£o..."
sleep 2

wget https://raw.githubusercontent.com/Stabillis/Stabillis_aplicacao/main/Aplicacao-Stabillis.jar
java -jar Aplicacao-Stabillis.jar
