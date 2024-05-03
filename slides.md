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

<div style="display:flex; align-items: center; justify-content:center; width: 100%">

<div style="width: 550px">

![Diagram](/images/diagram.jpg)
</div>
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

# Descrição dos componentes

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

# Descrição dos componentes

Explicação dos componentes dos diagramas.

## ⚒ Worker

|                    |                                                                                         |
| ------------------ | --------------------------------------------------------------------------------------- |
| <kbd>monitor</kbd> | Coleta métricas do worker e envia periodicmente ao controlador                          |
| <kbd>runner</kbd>  | Recebe instruções de deploy e inicia o processo correspondente no worker |

---
transition: slide-up
---

# Implementação Controller

## HTTP

* O controller usa um servidor HTTP para estabelecer comunicação com o workers e requests de SysAdmin. 

* Contém rotas especificas para cada request com seus "handlers" respectivos. 

---
transition: fade
level: 2
---

# Implementação Controller

## Deployer

* Lida com requisições de deploy dos operadores.
* Determina qual worker será utilizado para executar o serviço.
* Também lida com o teardown ("término") de serviços.

* Exemplo dos parâmetros de deploy:
```json
    // Domínio utilizado pelo balancer (cabeçalho HTTP "host")
    "name": "verde.pucminas.br",
    // Configurações de rede do serviço
    "network": {
        "expose_port": 80
    },
    // Imagem (mesmo conceito de imagens do Docker)
    "image": "gcr.io/...",
    // Quantas instâncias do serviço serão feitas
    "concurrency": 3
```

---
transition: fade
level: 2
---

# Implementação Controller

## Balancer

* Recebe requisições dos usuários e roteia para o serviço correspondente.

* Cabeçalho HTTP `Host`.

* Se mais de uma instância existir para um mesmo `Host`, fazemos balanceamento de carga.

* O balancer também faz terminação de TLS, de modo que os serviços não precisam se preocupar com isso.

---
transition: fade
level: 2
---

# Implementação Controller

## Discovery

* Peça crítica para a implementação de _service discovery_ no sistema.
    * Dinamicamente determina workers no sistema
    * Quais serviços (e em quais workers) estão disponíveis

* Database central que mantém informações sobre os workers, serviços, registros de deploys, métricas, etc.

* Baixa latência, mas alta confiabilidade e robustez do sistema mediante a falhas
    * Dados são persistidos ao disco utilizando SQLite
    * Modelo majoritariamente eventualmente consistente

---
transition: fade
level: 1
---

# Implementação Worker

## Collector

* Coleta informações da máquina worker, como uso da CPU e memória.

* Essas informaçõe são enviadas para o controller, com o objetivo de oferecer insights para balancear a carga.

---
transition: fade
---

# Progresso do Projeto

* Documentação Concluída
* Diagrama de Deploy
* Componentes do Worker `2/3`
* Componentes do Controller `3/5`

<br/>
<br/>

> 70% - Projeto Concluído

<div style="width: 100%; background-color: #1B1B1B; height: 32px; border: solid 1px #313131; border-radius: 5px; margin-top: 8px">
<div style="width: 70%; background-color: #5D8392; height: 100%; border-radius: 5px">
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

# Obrigado

[GitHub](https://github.com/esfericos/tucano)
