---
theme: seriph
background: https://cover.sli.dev
title: Tucano
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

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/esfericos/tucano" target="_blank" alt="GitHub" title="Github"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
---

# Indíce
<Toc maxDepth="1"></Toc>

---
layout: default
---

# O que é Tucano?


<v-click>
⏰ Simples <span v-mark.red="1">scheduler de seviços</span> capaz de gerenciar workloads diversos em um sistema composto por vários computadores.

<br/>
<br/>
<v-click>
⚖ Além disso, o scheduler também é responsavel pelo <span v-mark.red="2">balanceamento de carga</span>, de modo a fazer um service discovery para rotear requisições de usuários ao seus respectivos serviços.
</v-click>
</v-click>


---
transition: fade-out
---

# Diagrama

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

# Descrição dos componentes.
Explicação dos componentes dos diagramas.


## 🎮 Controller

|     |     |
| --- | --- |
| <kbd>deployer</kbd>| Aceita a configuração estática de um serviço e inicia o processo de deploy |
| <kbd>balancer</kbd> | Balaceia a carga aos nós correspondentes  |
| <kbd>agent_mgr</kbd> | Recebe informações dos agents e lida com eventuais "mortes" de workers. |
| <kbd>discovery</kbd> | Mantém informações necessárias para realizar service discovery. |


---
transition: slide-left
---


# Descrição dos componentes.
Explicação dos componentes dos diagramas.


## ⚒ Worker

|     |     |
| --- | --- |
| <kbd>monitor</kbd>| Coleta métricas do worker e envia periodicmente ao controlador |
| <kbd>runner</kbd> | Recebe instruções de deploy do controlador e inicia o processo correspondente no worker  |


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
  port: 80           | especifica qual porta deve ser aberta
  concurrency: 3     | quantidade de processos que devem estar executando o serviço 
  image: ...         | referência à imagem utilizada pelo processo
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
class: px-20
---

# Cronograma

Slidev comes with powerful theming support. Themes can provide styles, layouts, components, or even configurations for tools. Switching between themes by just **one edit** in your frontmatter:

<div grid="~ cols-2 gap-2" m="t-2">

```yaml
---
theme: default
---
```

```yaml
---
theme: seriph
---
```

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true" alt="">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true" alt="">

</div>

Read more about [How to use a theme](https://sli.dev/themes/use.html) and
check out the [Awesome Themes Gallery](https://sli.dev/themes/gallery.html).

---
layout: center
class: text-center
---

# Obrigado!
[GitHub](https://github.com/esfericos/tucano)
