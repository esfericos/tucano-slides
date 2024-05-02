---
theme: seriph
background: https://cover.sli.dev
title: Tucano
colorSchema: dark
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Tucano

why use one computer if u can use 5 with Tucano lol


<p style="font-size:14px; margin: 0px; margin-top: 32px">Eduardo Lemos Paschoalini</p>
<p style="font-size:14px; margin: 0px;">Gustavo Dias Aguiar</p>
<p style="font-size:14px; margin: 0px;">Luiz Felipe Frazão</p>
<p style="font-size:14px; margin: 0px;">Yuri Rousseff</p>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/esfericos/tucano" target="_blank" alt="GitHub" title="Github"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
---
<!-- END OF TUCANO -->

# Indíce

<Toc maxDepth="1"></Toc>

---
layout: default
---
<!-- END OF INDICE -->

# O que é Tucano?

<v-click>
⏰ Simples <span v-mark.red="1">scheduler de seviços</span> capaz de gerenciar workloads diversos em um sistema composto por vários computadores.

<br/>
<br/>
<v-click>
⚖ Além disso, o scheduler também é responsavel pelo <span v-mark.red="2">balanceamento de carga</span>, de modo a fazer um service discovery para rotear requisições de usuários ao seus respectivos serviços.
<br/>
<br/>
<v-click>
🖥 Uso de multiplos computadores físicos em rede local para <span v-mark.red="3">uso distribuido</span>
</v-click>
</v-click>
</v-click>

---
transition: fade-out
---
<!-- END OF "O QUE E TUCANO" -->

# Atualizações
<div style="display:flex; align-items: center; justify-content:center">

```mermaid {scale: 0.5, alt: 'A simple sequence diagram', theme: 'dark'}
graph TB
    user
    user -- request --> balancer

    sysadmin
    sysadmin -- (http) configures --> deployer
    sysadmin ---> config_mgr
    agent_mgr -- alerts --> sysadmin

    subgraph system-network

        subgraph ctrl
            deployer
            balancer
            agent_mgr
            config_mgr
            discovery

            deployer --> discovery
            discovery --- balancer
            agent_mgr --- discovery
        end

        balancer -- routes requests --> program
        deployer -- (http) deploy service --> runner
        monitor -- (http) send metrics and status --> agent_mgr

        subgraph worker
            monitor
            runner
            program

            runner --> program
        end

    end
```
</div>

---
transition: slide-up
level: 2
---

# Atualizações
<div style="display:flex; align-items: center; justify-content:center">

```mermaid {scale: 0.5, alt: 'A simple sequence diagram', theme: 'dark'}
graph TB
    user
    user -- request --> balancer

    sysadmin
    sysadmin -- (http) configures --> deployer
    sysadmin ---> config_mgr
    agent_mgr -- alerts --> sysadmin

    subgraph system-network

        subgraph ctrl
            deployer
            balancer
            agent_mgr
            config_mgr
            discovery

            deployer --> discovery
            discovery --- balancer
            agent_mgr --- discovery
        end

        balancer -- routes requests --> program
        deployer -- (http) deploy service --> runner
        monitor -- (http) send metrics and status --> agent_mgr

        subgraph worker
            monitor
            runner
            program

            runner --> program
        end

    end
```
</div>

---
transition: slide-up
level: 2
---











# Atualizações

<div style="display:flex; align-items: center; justify-content:center">

```mermaid {scale: 0.3, alt: 'Deployment Sequency Diagram', theme: 'dark'}
sequenceDiagram
    %% Alice->>+John: Hello John, how are you?
    %% Alice->>+John: John, can you hear me?
    %% John-->>-Alice: Hi Alice, I can hear you!
    %% John-->>-Alice: I feel great!
    actor SysAdmin
    participant deployer as ctl::deployer
    participant discovery as ctl::discovery
    participant wrk_mgr as ctl::worker_mgr

    participant runner as wrk::runner
    participant builder as wrk::builder
    participant supervisor as wrk::supervisor

    SysAdmin ->>+ deployer: New deploy request (via CLI)
    deployer ->>+ discovery: Create new deployment
    discovery ->>- deployer: Deployment ID
    deployer ->>- SysAdmin: Deployment ID

    deployer ->>+ discovery: Fetch available workers
    discovery ->>- deployer: 

    deployer ->> deployer: Select worker

    deployer -->>+ runner: Start deployment

    runner ->>+ builder: Execute build script
    builder ->>- runner: Report status

    alt build failed
        runner -->> wrk_mgr: Report build failure
        wrk_mgr -->> discovery: Record build failed status
    else build ok
        runner ->>+ supervisor: Start service
        supervisor ->>- runner: Report status

        alt service started
            runner -->> wrk_mgr: Report service running
            wrk_mgr -->> discovery: Record running status

            opt service crashed
                runner -->> wrk_mgr: Report failed
                wrk_mgr -->> discovery: Record failed status
            end
        else service failed to start
            runner -->> wrk_mgr: Report failure
            wrk_mgr -->> discovery: Record failed status
        end
    end
```
</div>

---
transition: slide-up
level: 2
---











<!-- END OF ATUALIZACOES -->

# Descrição dos componentes.

Explicação dos componentes dos diagramas.

## 🎮 Controller

|                      |                                                                            |
| -------------------- | -------------------------------------------------------------------------- |
| <kbd>http*</kbd>      | Recebe requisições externas dos workers ou admin e roteia para o componente correspondente |
| <kbd>deployer</kbd>  | Aceita a configuração estática de um serviço e inicia o processo de deploy |
| <kbd>balancer</kbd>  | Balaceia a carga aos nós correspondentes                                   |
| <kbd>worker_mgr</kbd>| Recebe informações dos agents e lida com eventuais "mortes" de workers.    |
| <kbd>discovery</kbd> | Mantém informações necessárias para realizar service discovery.            |

---
transition: slide-left
---

# Descrição dos componentes.

Explicação dos componentes dos diagramas.

## ⚒ Worker

|                    |                                                                                         |
| ------------------ | --------------------------------------------------------------------------------------- |
| <kbd>monitor</kbd> | Coleta métricas do worker e envia periodicmente ao controlador                          |
| <kbd>runner</kbd>  | Recebe instruções de deploy e inicia o processo correspondente no worker |

---
transition: slide-up
---

# Na prática

Queremos lançar nosso cardápio digítal usando Tucano.

<v-click>

## Processo de deploy

<v-click>

1. Vamos definir algumas regras no nosso arquivo de configuração

```md
port: 80 | especifica qual porta deve ser aberta
concurrency: 3 | quantidade de processos que devem estar executando o serviço
image: ... | referência à imagem utilizada pelo processo
```

<div v-click style="margin-top: 24px">

2. Definir o `build script`

```md
yarn build ...
```

<div v-click="4" style="margin-top: 24px">

3. Definir o `runtime script`

```md
yarn run ...
```

</div>
</div>

</v-click>
</v-click>

---
transition: fade
level: 2
---

# Na prática

Queremos lançar nosso cardápio digítal usando Tucano.

## Runtime

<div
  style="
  display: flex;
  flex-direction: row;
  width: 100%;
  margin-top: 24px;
  "
>

<div v-click="1"
  style="margin-right: 90px"
>
<p
  style="
  font-size: 10px;
  margin: 0px;
  color: gray;
  font-weight: 600;"> Requisição de usuário </p>

```mermaid {scale: 0.6, alt: 'User Request', theme: 'dark'}
graph TB
    user
    user -- request --> balancer

    subgraph system-network

        subgraph ctrl
            balancer
        end

        balancer -- routes requests --> program

        subgraph worker
            program
        end

    end
```

</div>

<div style="display: flex; align-items: center">
<div v-click="2" style="margin-right: 16px" >
<p
  style="
  font-size: 10px;
  margin: 0px;
  color: gray;
  margin-left: 32px;
  font-weight: 600;">Balancer</p>
  
```mermaid {scale: 0.4, alt: 'User Request', theme: 'dark'}
graph TB
    user -- request --> balancer
    subgraph system-network

        subgraph ctrl
            balancer
            agent_mgr
            discovery

            discovery --- balancer
            agent_mgr --- discovery
        end

        balancer -- routes requests --> program
        monitor -- (http) send metrics and status --> agent_mgr

        subgraph worker
            monitor
            program

        end

    end

```
</div>

<div v-click="3">

Disponibilidade decidida a partir de métricas de consumo do worker (consumo de CPU, memória, disco, etc...) e enviadas para o `agent_mgr`

</div>
</div>
</div>



---
transition: fade
---

# Metodologia

<div
  style="display: flex; align-items: flex-start; width: 100%; justify-content: space-between"
>

<blockquote v-click="1" style="width: 50%; transition: all 350ms ease;">

**📆 Reuniões semanais**

 <hr style="border-top: solid 1px gray; padding: 4px; margin-top: 8px"/>

 <p style="font-size:12px">Domingo, 09:00h - 12:00h</p>

 <p style="font-size:12px">Sexta-Feira, 10:40h - 12:20h</p>

 <p style="font-size:12px"> Rever progresso, discutir desafios, estabelecer metas</p>

 <p style="font-size:12px"> Alinhamento de prioridades e distribuição de tarefas</p>
</blockquote>

<blockquote v-click="2" style="width: 45%; transition: all 350ms ease;">

 **🔰 Reuniões Ocasionais**

 <hr style="border-top: solid 1px gray; padding: 4px; margin-top: 8px"/>

 <p style="font-size:12px">Ajustes rápidos, discussões emergenciais</p>

 <p style="font-size:12px"> Alinhamento de prioridades e distribuição de tarefas</p>

 <p style="font-size:12px">Sessões de Pair Programming</p>
</blockquote>

</div>

---
layout: center
class: text-center
---

# Obrigado!
[GitHub](https://github.com/esfericos/tucano)
