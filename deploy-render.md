# Guia de Deploy no Render - Kanban System JWT

Este guia detalha o passo a passo necessário para realizar o deploy com sucesso da aplicação **Kanban System JWT** na plataforma [Render](https://render.com).

As alterações de código necessárias para tornar o projeto compatível com a produção (leitura dinâmica de portas e variável de ambiente para o segredo do JWT) já foram realizadas no servidor.

---

## 1. Alterações Estruturais Realizadas no Projeto

Para garantir que a aplicação funcione corretamente em ambiente de produção na nuvem, foram realizadas as seguintes padronizações:

### Servidor (`server.js`)
* **Porta Dinâmica**: Alterado de porta estática `3000` para `process.env.PORT || 3000`. O Render injeta automaticamente a variável `PORT` na máquina onde o serviço é executado.
* **Segurança do JWT**: A chave secreta do JWT agora utiliza a variável de ambiente `process.env.JWT_SECRET` com um fallback seguro para desenvolvimento local:
  ```javascript
  const SECRET_KEY = process.env.JWT_SECRET || 'super-secret-key-for-jwt-do-not-use-in-prod';
  ```

### Front-end (`public/index.html`)
* **URL de API Dinâmica**: O endereço de comunicação com a API foi alterado para caminhos relativos. Se o arquivo for aberto diretamente no navegador (via `file://`), ele tentará se comunicar com o `localhost:3000`; caso contrário, fará requisições relativas no mesmo domínio do servidor (evitando erros de CORS em produção):
  ```javascript
  const API_URL = window.location.protocol === 'file:' ? 'http://localhost:3000/api' : '/api';
  ```

---

## 2. Passo a Passo do Deploy no Render

Siga as etapas abaixo para publicar a aplicação:

### Passo 1: Enviar as Atualizações para o GitHub
Certifique-se de realizar o commit e push de todas as alterações feitas localmente para o seu repositório remoto do GitHub:
```bash
git add .
git commit -m "chore: preparar projeto para deploy no Render"
git push origin main
```

### Passo 2: Criar um Novo Web Service no Render
1. Acesse o dashboard do [Render](https://render.com) e faça login (recomenda-se login com GitHub).
2. Clique no botão **"New +"** no canto superior direito e selecione **"Web Service"**.
3. Conecte sua conta do GitHub caso ainda não tenha feito.
4. Na lista de repositórios, localize e selecione o repositório: `newkanbansabadoprof2`.

### Passo 3: Configurar os Parâmetros do Web Service
Preencha os campos de configuração com os seguintes valores:

| Campo | Valor Recomendado | Observação |
| :--- | :--- | :--- |
| **Name** | `newkanbansabado` *(ou de sua escolha)* | Nome que comporá a URL pública (ex: `newkanbansabado.onrender.com`). |
| **Region** | `Oregon (US West)` ou `Frankfurt (EU Central)` | Escolha a região geográfica mais próxima do seu público. |
| **Branch** | `main` | O branch que contém a versão estável do projeto. |
| **Root Directory** | *Deixar em branco* | Os arquivos de configuração (`package.json`) estão na raiz do repositório. |
| **Runtime** | `Node` | Linguagem/ambiente de execução da aplicação. |
| **Build Command** | `npm install` | Comando executado para instalar as dependências de produção. |
| **Start Command** | `npm start` | Comando para iniciar o servidor Express. |

### Passo 4: Configurar as Variáveis de Ambiente (Environment Variables)
1. Role a página de configurações até a seção **"Environment Variables"** (ou clique na aba *Env Groups*).
2. Adicione as seguintes variables clicando em **"Add Environment Variable"**:

| Key | Value | Descrição |
| :--- | :--- | :--- |
| **`NODE_ENV`** | `production` | Informa à aplicação e bibliotecas que ela está rodando em produção. |
| **`JWT_SECRET`** | *[Insira uma chave secreta forte]* | Uma string longa e aleatória usada para encriptar os tokens JWT de segurança dos usuários. *(Ex: `6f9c9c3e414bd65839b81b8979e2c650db8167f9`)* |

> **Atenção:** A porta do servidor (`PORT`) **não** precisa ser configurada manualmente. O Render a define automaticamente e o seu código já a lê usando `process.env.PORT`.

### Passo 5: Iniciar o Deploy
1. Escolha o plano **"Free"** (gratuito) para fins de teste/aprendizado ou planos superiores conforme sua necessidade.
2. Clique em **"Create Web Service"**.
3. O Render iniciará o processo de build (instalação das dependências) e, em seguida, executará o servidor. Acompanhe os logs no console do painel.
4. Assim que o status mudar para **"Live"**, sua aplicação estará online na URL fornecida no topo da tela!

---

## 3. Limitações Importantes do Plano Gratuito (Free Tier) e Armazenamento

> **Importante:**
> **Sistema de Arquivos Efêmero (Perda de Dados)**
> Este projeto utiliza arquivos JSON locais (`users.json`, `tasks.json`, `revoked_tokens.json`) como banco de dados. 
> Nos servidores em nuvem do Render (especialmente no plano gratuito), o disco do servidor é **efêmero**. Isso significa que **toda vez que o servidor for reiniciado, atualizado ou entrar em modo de suspensão por inatividade, os novos usuários criados e as novas tarefas adicionadas serão perdidos**, resetando para o estado inicial contido nos arquivos enviados ao GitHub.

### Como Resolver a Persistência de Dados em Produção:

1. **Uso de Persistent Disks (Pago)**:
   * No Render, é possível adicionar um disco de armazenamento persistente (*Persistent Disk*) nas configurações do serviço.
   * Ao fazer isso, configure o ponto de montagem (ex: `/data`) e atualize os caminhos dos arquivos no `server.js` para apontar para a pasta `/data` (ex: `/data/users.json`).
   
2. **Migração para Banco de Dados na Nuvem (Recomendado)**:
   * Para projetos de produção reais, substitua a leitura de arquivos JSON locais por um banco de dados como o **MongoDB Atlas** (plano gratuito disponível) ou **PostgreSQL** hospedado na própria plataforma Render.
