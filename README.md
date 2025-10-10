# 🐳 Laboratório Docker – Fundamentos e Prática com Containers

> **Objetivo:** Compreender os conceitos fundamentais de **containers**, dominar os principais comandos do **Docker**, entender modos de execução, identificação de containers e implantar um **servidor web NGINX** customizado.

---

## 🧠 1. Conceitos Fundamentais sobre Containers

### O que são Containers?

Containers são **unidades padronizadas de software** que empacotam o código e todas as suas dependências (bibliotecas, configurações, binários), garantindo que uma aplicação seja executada de forma consistente em qualquer ambiente.

Ao contrário de uma **máquina virtual**, que virtualiza um sistema operacional completo com seu próprio kernel, um **container compartilha o kernel do host**, tornando-o mais leve, rápido e eficiente.

#### Containers vs. Máquinas Virtuais

| Característica            | Máquina Virtual (VM)                | Container (Docker)                     |
|---------------------------|-------------------------------------|----------------------------------------|
| Virtualização             | Hardware completo (via hypervisor)  | Nível de sistema operacional           |
| Tempo de inicialização    | Minutos                             | Segundos (ou milissegundos)            |
| Tamanho típico            | Vários GB (5-20 GB)                 | Centenas de MB ou menos                |
| Isolamento                | Total (SO próprio)                  | Parcial (namespaces e cgroups)         |
| Eficiência de recursos    | Menor (overhead significativo)      | Maior (compartilha kernel)             |
| Densidade                 | Dezenas por host                    | Centenas por host                      |
| Portabilidade             | Limitada (formato específico)       | Alta (OCI padrão)                      |

### Por que usar Containers?

- **Portabilidade:** O mesmo container roda em qualquer ambiente com Docker (dev, teste, produção)
- **Rapidez:** Inicialização quase instantânea comparada a VMs
- **Isolamento:** Cada aplicação roda em seu próprio ambiente isolado
- **Escalabilidade:** Fácil de replicar e distribuir horizontalmente
- **Reprodutibilidade:** Elimina o clássico "funciona na minha máquina"
- **Eficiência:** Melhor aproveitamento de recursos computacionais
- **Versionamento:** Imagens podem ser versionadas e controladas
- **Microsserviços:** Arquitetura ideal para aplicações distribuídas

### Arquitetura do Docker

```
┌─────────────────────────────────────────────┐
│         Cliente Docker (CLI)                │
│      docker run, docker build, etc.         │
└──────────────────┬──────────────────────────┘
                   │ API REST
┌──────────────────▼──────────────────────────┐
│          Docker Daemon (dockerd)            │
│  ┌────────────┐  ┌────────────┐             │
│  │ Containers │  │  Imagens   │             │
│  └────────────┘  └────────────┘             │
│  ┌────────────┐  ┌────────────┐             │
│  │   Volumes  │  │   Redes    │             │
│  └────────────┘  └────────────┘             │
└─────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│          Kernel do Sistema Operacional      │
└─────────────────────────────────────────────┘
```

---

## ⚙️ 2. Pré-requisitos

Você pode realizar este laboratório de duas formas:

### 🧩 Opção 1 – Ambiente Online (sem instalação)
1. Crie uma conta gratuita no **Docker Hub**: [https://hub.docker.com](https://hub.docker.com)
2. Acesse **[Play With Docker (PWD)](https://labs.play-with-docker.com)**  
   - Clique em **"Start"**  
   - Aguarde a abertura de um terminal Linux online (válido por 2h)
   - **Limitações:** Sem persistência após 4h, recursos limitados

### 💻 Opção 2 – Docker Desktop (Windows/Mac/Linux)
1. **Windows/Mac:** Baixe o **Docker Desktop**:  
   [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
   
2. **Linux (Ubuntu/Debian):**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   # Reinicie a sessão para aplicar permissões
   ```

3. **Verifique a instalação:**
   ```bash
   docker version
   docker info
   ```

---

## 🎯 3. Conceitos Essenciais: Identificação e Modos de Execução

### 🔑 Identificação de Containers

Todo container Docker pode ser identificado de **três formas diferentes**:

#### 1. Container ID (Hash Completo)
- Hash SHA256 de 64 caracteres
- Exemplo: `a3f5b8c7e9d1f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2`
- Raramente usado na prática (muito longo)

#### 2. Short ID (Hash Curto)
- Primeiros 12 caracteres do hash completo
- Exemplo: `a3f5b8c7e9d1`
- **Mais comum** para identificação rápida
- Obtido com: `docker ps`

#### 3. Nome do Container
- Nome legível atribuído automaticamente ou manualmente
- Exemplo automático: `loving_tesla`, `gallant_turing`
- Exemplo manual: `meu-webserver`, `api-backend`
- Atribuído com: `--name nome_do_container`

**Exemplo prático:**
```bash
# Criar container com nome customizado
docker run -d --name meu-nginx nginx

# Criar container sem nome (nome automático será gerado)
docker run -d nginx

# Listar e ver as diferentes formas de identificação
docker ps
# CONTAINER ID   IMAGE   COMMAND                  NAMES
# a3f5b8c7e9d1   nginx   "nginx -g 'daemon of…"   meu-nginx
# b7e2c4f6a8d0   nginx   "nginx -g 'daemon of…"   romantic_fermi
```

**Importante:** Você pode usar qualquer uma das três formas para gerenciar containers:
```bash
docker stop a3f5b8c7e9d1        # Usando Short ID
docker stop meu-nginx           # Usando nome
docker logs romantic_fermi      # Usando nome automático
```

---

### 🔄 Modos de Execução: Interativo vs. Detached

O Docker oferece diferentes modos de executar containers, cada um adequado para cenários específicos:

#### 🖥️ Modo Interativo (`-it`)

O modo interativo mantém uma **sessão ativa** com o container, conectando seu terminal ao STDIN/STDOUT do processo.

**Flags:**
- `-i` (--interactive): Mantém STDIN aberto mesmo sem conexão
- `-t` (--tty): Aloca um pseudo-TTY (terminal)
- Geralmente usados juntos: `-it`

**Quando usar:**
- Executar comandos dentro do container em tempo real
- Debugging e exploração
- Aplicações que requerem input do usuário
- Shells interativos (bash, sh, etc.)

**Exemplo:**
```bash
# Executar um container Ubuntu com shell interativo
docker run -it ubuntu bash

# Você estará DENTRO do container
root@a3f5b8c7e9d1:/# ls
root@a3f5b8c7e9d1:/# pwd
root@a3f5b8c7e9d1:/# exit  # Sai e para o container
```

**Características:**
- ✅ Terminal fica "preso" ao container
- ✅ Você vê output em tempo real
- ✅ Pode interagir com o processo
- ❌ Se fechar o terminal, o container para
- ❌ Não ideal para serviços de longa duração

---

#### 🔌 Modo Detached (`-d`)

O modo *detached* executa o container em **segundo plano**, liberando o terminal.

**Flag:**
- `-d` (--detach): Executa container em background

**Quando usar:**
- Serviços web (NGINX, Apache)
- Bancos de dados (MySQL, PostgreSQL)
- APIs e microsserviços
- Qualquer aplicação que deve rodar continuamente
- Ambientes de produção

**Exemplo:**
```bash
# Executar NGINX em background
docker run -d --name webserver -p 8080:80 nginx

# O terminal retorna imediatamente com o Container ID
a3f5b8c7e9d1f2a4b6c8d0e2f4a6b8c0

# Container continua rodando em background
docker ps
# CONTAINER ID   STATUS    PORTS                  NAMES
# a3f5b8c7e9d1   Up 5s     0.0.0.0:8080->80/tcp   webserver
```

**Características:**
- ✅ Terminal fica livre para outros comandos
- ✅ Container continua rodando mesmo se fechar o terminal
- ✅ Ideal para serviços de longa duração
- ✅ Pode gerenciar múltiplos containers simultaneamente
- ❌ Não vê output direto (usar `docker logs`)

---

#### 🔀 Combinando Modos: Detached + Interativo

Você pode iniciar um container em detached e depois conectar interativamente:

```bash
# 1. Iniciar em detached
docker run -d --name ubuntu-server ubuntu sleep infinity

# 2. Conectar interativamente depois
docker exec -it ubuntu-server bash

# Agora você está dentro, mas ao sair (exit),
# o container continua rodando!
```

---

#### 📊 Comparação Prática

| Aspecto                  | Modo Interativo (`-it`)     | Modo Detached (`-d`)        |
|--------------------------|-----------------------------|-----------------------------|
| Terminal                 | Conectado ao container      | Livre para outros comandos  |
| Visibilidade de logs     | Imediata no terminal        | Via `docker logs`           |
| Ao fechar terminal       | Container para              | Container continua          |
| Uso típico               | Debug, testes, shells       | Serviços, produção          |
| Exemplo                  | `docker run -it ubuntu`     | `docker run -d nginx`       |
| Input do usuário         | ✅ Possível                  | ❌ Não disponível            |
| Múltiplos containers     | Difícil (1 terminal/cont.)  | Fácil (todos em background) |

---

## 🧩 4. Laboratório Prático

**⚠️ Importante:** Durante a execução, registre cada comando e observação em um arquivo `comandos.txt`.

---

### 📹 Passo 1 – Executar o container "Hello World"

```bash
docker run hello-world
```

**O que acontece:**
1. Docker procura a imagem localmente
2. Não encontra, faz download do Docker Hub
3. Cria e executa o container
4. Exibe mensagem de confirmação
5. Container finaliza automaticamente

**Análise:**
```bash
# Verificar que o container foi criado mas está parado
docker ps -a

# Você verá algo como:
# CONTAINER ID   STATUS                     NAMES
# abc123def456   Exited (0) 2 minutes ago   reverent_morse
```

---

### 📹 Passo 2 – Criar um servidor web NGINX (modo detached)

```bash
docker run -d --name webserver -p 8080:80 nginx
```

**Detalhamento das flags:**
- `-d`: Modo **detached** (background)
- `--name webserver`: Nome customizado para facilitar gerenciamento
- `-p 8080:80`: **Port mapping** (host:container)
  - `8080`: Porta no seu computador (host)
  - `80`: Porta dentro do container
- `nginx`: Nome da imagem a ser usada

**Acessar o servidor:**
- **Play With Docker:** Clique em "OPEN PORT" → Digite `8080`
- **Docker Desktop:** Acesse [http://localhost:8080](http://localhost:8080)

**Você deve ver:** A página padrão "Welcome to nginx!"

---

### 📹 Passo 3 – Listar containers (entendendo os estados)

```bash
# Containers em execução (running)
docker ps

# TODOS os containers (running + stopped)
docker ps -a

# Formato customizado e mais legível
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

# Apenas os IDs dos containers em execução
docker ps -q

# Último container criado
docker ps -l
```

**Estados possíveis:**
- `Up`: Container rodando
- `Exited`: Container parado
- `Created`: Container criado mas nunca iniciado
- `Restarting`: Container reiniciando
- `Paused`: Container pausado

---

### 📹 Passo 4 – Trabalhar com logs

```bash
# Ver todos os logs do container
docker logs webserver

```


---

### 📹 Passo 5 – Modo interativo: Acessar e modificar o container

```bash
# Conectar ao container em modo interativo
docker exec -it webserver bash

# Agora você está DENTRO do container!
# O prompt mudará para algo como: root@a3f5b8c7e9d1:/#

# Ver o conteúdo padrão do NGINX
cat /usr/share/nginx/html/index.html

# Modificar a página principal
echo '<h1>🐳 Docker é fácil!</h1><p>Modificado em tempo real!</p>' > /usr/share/nginx/html/index.html

# Sair do container (ele continua rodando)
exit
```

**Recarregue o navegador** → Você verá a página modificada!


---

### 📹 Passo 6 – Gerenciar ciclo de vida do container

```bash
# Parar o container (graceful shutdown)
docker stop webserver

# Verificar status
docker ps -a
# STATUS: Exited (0) X seconds ago

# Iniciar novamente
docker start webserver

# Verificar status
docker ps
# STATUS: Up X seconds

# Reiniciar (stop + start)
docker restart webserver

# Pausar (congela o processo)
docker pause webserver

# Despausar
docker unpause webserver

# Forçar parada imediata (não recomendado)
docker kill webserver
```

**Diferença entre stop e kill:**
- `stop`: Envia SIGTERM, aguarda 10s, depois SIGKILL (graceful)
- `kill`: Envia SIGKILL imediatamente (forçado)



---

### 📹 Passo 7 – Remover containers e imagens

```bash
# Parar containers em execução
docker stop webserver

# Remover containers específicos
docker rm webserver 

# Remover container à força (sem stop)
docker rm -f container_name

# Remover TODOS os containers parados
docker container prune

# Listar imagens baixadas
docker images

# Remover imagens específicas
docker rmi nginx busybox hello-world

# Remover imagens não utilizadas
docker image prune

# CUIDADO: Remover tudo (containers, imagens, volumes, redes)
docker system prune -a --volumes
```

---

## 🌐 5. Atividade Final – Servidor Web Personalizado

Nesta atividade, você irá implantar uma aplicação web customizada usando NGINX e volumes, servindo o conteúdo de um repositório Git.

### 📋 Passos da Atividade

#### 1. Clonar o repositório de exemplo

```bash
# Clonar repositório com conteúdo web
git clone https://github.com/fatec-cd/pratica-docker.git
cd pratica-docker
```

**📝 Nota para Windows:**
- Se não tiver Git instalado, baixe em: [https://git-scm.com/download/win](https://git-scm.com/download/win)
- Ou baixe o ZIP do repositório diretamente no GitHub e extraia

#### 2. Executar NGINX com bind mount

**Linux/Mac/Play With Docker:**
```bash
docker run -d --name meuweb \
  -p 8080:80 \
  -v $(pwd):/usr/share/nginx/html:ro \
  nginx
```

**Windows (PowerShell):**
```bash
docker run -d --name meuweb -p 8080:80 -v ${PWD}:/usr/share/nginx/html:ro nginx
```

**Windows (CMD):**
```bash
docker run -d --name meuweb -p 8080:80 -v %cd%:/usr/share/nginx/html:ro nginx
```

#### 3. Entendendo o comando

**Detalhamento das flags:**
- `-d`: Modo detached (background)
- `--name meuweb`: Nome do container
- `-p 8080:80`: Mapeia porta 8080 (host) → 80 (container)
- `-v`: **Bind mount** - conecta diretório do host ao container
  - `$(pwd)` ou `${PWD}`: Diretório atual (onde está o repositório)
  - `/usr/share/nginx/html`: Diretório padrão do NGINX no container
  - `:ro`: **Read-only** - container só pode ler, não modificar
- `nginx`: Imagem oficial do NGINX

#### 4. Acessar a aplicação

- **Play With Docker:** Clique no botão **"OPEN PORT"** → Digite `8080`
- **Docker Desktop:** Acesse [http://localhost:8080](http://localhost:8080)

**✅ Resultado esperado:** Você deve visualizar a página HTML do repositório sendo servida pelo NGINX.


#### 5. Limpar o ambiente

```bash
# Parar e remover o container
docker stop meuweb
docker rm meuweb

# O diretório do repositório permanece intacto!
ls pratica-docker/
```

---

## 📤 6. Entrega da Atividade

Envie no **Microsoft Teams** 

1. **Screenshots** :
   - Página do NGINX padrão rodando
   - Página modificada (Passo 5)
   - Atividade final em execução

2. **`respostas.txt`** com as questões abaixo:

#### Questões obrigatórias:

**Q1.** Qual a diferença entre `docker ps` e `docker ps -a`? Qual comando mostra apenas containers finalizados?

**Q2.** Explique a diferença entre executar um container em modo interativo (`-it`) e em modo detached (`-d`). Cite dois exemplos de uso para cada modo.

**Q3.** Cite três vantagens de executar containers em modo detached.

**Q4.** Como um container pode ser identificado? Explique as três formas e quando usar cada uma.


---

## 🎓 7. Conceitos Extras para Estudo

### 🔹 Boas Práticas

1. **Use nomes descritivos:** `--name api-backend` melhor que `--name test1`
2. **Sempre especifique versões de imagens:** `nginx:1.25` melhor que `nginx:latest`
3. **Use volumes para dados persistentes:** Nunca confie em dados dentro do container
4. **Minimize camadas em Dockerfiles:** Combine comandos quando possível
5. **Use .dockerignore:** Evite copiar arquivos desnecessários
6. **Rode containers como non-root:** Aumente a segurança
7. **Limpeza regular:** Use `docker system prune` periodicamente

### 🔹 Troubleshooting Comum

| Problema | Solução |
|----------|---------|
| "Port already allocated" | Outra aplicação usando a porta. Use porta diferente ou `docker stop` no container conflitante |
| "Cannot connect to Docker daemon" | Docker Desktop não está rodando ou usuário sem permissão |
| Container para imediatamente | Processo principal terminou. Use `docker logs` para investigar |
| "No such file or directory" em volumes | Caminho incorreto. Use caminho absoluto ou `$(pwd)` |
| Imagem não encontrada | Verifique o nome. Use `docker pull` primeiro |

---

## 🧩 8. Referências e Recursos

### 📚 Documentação Oficial
- [Docker Documentation](https://docs.docker.com/) - Documentação completa e atualizada
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/) - Todos os comandos
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/) - Sintaxe do Dockerfile
- [Docker Hub](https://hub.docker.com/) - Repositório oficial de imagens

### 🎓 Tutoriais e Cursos
- [Play With Docker Classroom](https://training.play-with-docker.com/) - Tutoriais interativos gratuitos
- [Docker Labs](https://github.com/docker/labs) - Repositório oficial de laboratórios
- [Docker Curriculum](https://docker-curriculum.com/) - Guia para iniciantes
- [Katacoda Docker Scenarios](https://www.katacoda.com/courses/docker) - Cenários práticos

### 📖 Livros e Guias
- "Docker Deep Dive" - Nigel Poulton
- "Docker in Action" - Jeff Nickoloff
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/) - Guia oficial

### 🛠️ Ferramentas Úteis
- [Docker Desktop](https://www.docker.com/products/docker-desktop) - Interface gráfica oficial
- [Portainer](https://www.portainer.io/) - Interface web para gerenciar Docker
- [Dive](https://github.com/wagoodman/dive) - Analisar camadas de imagens
- [Hadolint](https://github.com/hadolint/hadolint) - Linter para Dockerfiles

### 🌐 Comunidade
- [Docker Community Forums](https://forums.docker.com/)
- [Stack Overflow - Docker Tag](https://stackoverflow.com/questions/tagged/docker)
- [Reddit r/docker](https://www.reddit.com/r/docker/)
- [Docker Community Slack](https://www.docker.com/docker-community)

### 📺 Vídeos (em português)
- [Canal Full Cycle](https://www.youtube.com/@FullCycle) - Desenvolvimento com Docker
- [Canal Código Fonte TV](https://www.youtube.com/@codigofontetv) - Conceitos e práticas

---

## 🎯 9. Próximos Passos

Após dominar este laboratório, explore:

1. **Docker Compose** - Orquestrar múltiplos containers
2. **Dockerfile** - Criar suas próprias imagens customizadas
3. **Volumes nomeados** - Persistência de dados mais robusta
4. **Redes Docker** - Comunicação entre containers
5. **Docker Registry** - Criar registro privado de imagens
6. **Multi-stage builds** - Otimizar tamanho de imagens
7. **Docker Swarm** - Orquestração nativa do Docker
8. **Kubernetes** - Orquestração em escala empresarial


## 📋 10. Checklist de Conclusão

Antes de finalizar, verifique se você:

- [ ] Executou todos os comandos do laboratório
- [ ] Entendeu a diferença entre modo interativo e detached
- [ ] Compreendeu as formas de identificar containers (ID, short ID, nome)
- [ ] Conseguiu acessar o NGINX no navegador
- [ ] Modificou o conteúdo da página usando `docker exec`
- [ ] Criou o servidor web personalizado (atividade final)
- [ ] Capturou as screenshots necessárias
- [ ] Respondeu às questões em `respostas.txt`

---

🧭 **Boa prática e bons estudos!**

> *"Containers não são o futuro, são o presente. Dominar Docker é essencial para qualquer profissional de tecnologia moderno."*

---


