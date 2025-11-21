---
title: Entendendo PrivEsc em Kubernetes
published: 2025-01-20
tags: [Kubernetes, Segurança, Containers]
category: Cloud
draft: false
---

# Kubernetes e o Risco de PrivEsc

Kubernetes (K8s) se tornou uma das tecnologias mais usadas em ambientes cloud e aplicações modernas. Ele facilita o gerenciamento de containers, permite escalar serviços facilmente e ajuda empresas a rodar aplicações de forma mais eficiente. Mas, como toda tecnologia popular, ele também virou alvo de atacantes.

Um dos principais riscos dentro de um cluster Kubernetes é o **escalonamento de privilégios**, que acontece quando um invasor consegue ganhar mais permissões do que deveria e passa a controlar partes do cluster que deveriam estar protegidas.

Neste artigo, vamos ver, de forma simples, como isso acontece e quais os pontos mais atacados.

## Como Atacantes Tentam Escalar Privilégios

Depois de conseguir algum tipo de acesso dentro do cluster, o atacante normalmente tenta aumentar suas permissões. Isso pode ser feito explorando falhas, má configurações ou contas com permissões exageradas.

Algumas formas comuns de escalonamento:

### 1. Manipulação de Contas
O invasor tenta alterar permissões de contas existentes ou criar novas permissões para si mesmo.  
No Kubernetes, isso pode acontecer quando **Roles** e **ClusterRoles** estão muito abertas.

Exemplo comum:
- Criar um **RoleBinding** ligando uma conta fraca a um papel com permissões altas.

### 2. Uso de Contas Válidas
Às vezes, o ataque nem precisa quebrar nada.  
O invasor encontra ou rouba uma conta já existente e usa ela para continuar o ataque.

Isso inclui:
- Service Accounts padrão.
- Credenciais esquecidas em pods.
- Tokens expostos.

### 3. Falhas em Contêineres e Pods
Se um pod estiver configurado com privilégios demais, o invasor pode fugir do container e atacar o host.

Alguns erros comuns:
- Container rodando como **privileged**.
- Montar volumes sensíveis do host.
- Permitir comandos internos como se fosse o próprio nó.

### 4. RCE via Recursos do Cluster
Algumas permissões permitem executar comandos dentro de outros pods, o que abre muitas portas.

Permissões perigosas incluem:
- `create pods/exec`
- `update` ou `patch` em DaemonSets  
  (pode permitir espalhar containers maliciosos por toda a infraestrutura)

### 5. Jobs e CronJobs
Agendamentos também podem ser abusados.  
Se o atacante puder criar ou modificar Jobs e CronJobs, ele pode usar isso para rodar código malicioso no cluster repetidamente.

## RBAC: Onde Quase Tudo Acontece

Grande parte dos problemas de privilégio vem do RBAC mal configurado.

No Kubernetes, temos:

- **Roles** → permissões dentro de um namespace  
- **ClusterRoles** → permissões globais no cluster  
- **RoleBinding** → liga um Role a uma conta  
- **ClusterRoleBinding** → liga um ClusterRole a uma conta

O problema começa quando alguém dá permissões de mais ou liga uma conta simples a um papel poderoso.  
Isso cria o caminho perfeito para uma escalada de privilégios.

## System Pods: O Ponto Cego de Muita Gente

Os **system pods** são os pods que fazem o cluster funcionar. Eles são criados automaticamente pelo provedor ou pelo próprio Kubernetes.

O risco é que:
- Eles geralmente têm privilégios muito altos.
- Nem sempre os usuários conseguem vê-los ou configurá-los.
- Qualquer falha neles pode virar um ataque grave.

Atacantes sabem disso e procuram formas de abusar desses pods ou de pods localizados no mesmo nó.

## Misconfiguration: O Verdadeiro Problema

A maioria dos ataques em Kubernetes não começa com uma falha zero-day.  
Começa com **configurações erradas**:

- Roles muito permissivas.
- Service Accounts com acesso demais.
- Volumes montados sem necessidade.
- Pods privilegiados.
- Falta de segmentação entre namespaces.

Quando várias dessas falhas se juntam, o invasor consegue montar uma “cadeia de ataque” e, passo a passo, tomar o cluster inteiro.

## Conclusão

O Kubernetes é poderoso, mas também complexo, e por isso erros simples podem criar brechas enormes. Escalar privilégios é um dos ataques mais comuns e mais perigosos dentro de clusters, e acontece principalmente por má configuração e permissões exageradas.

Entender como RBAC funciona, revisar permissões e observar system pods são passos essenciais para manter um ambiente Kubernetes seguro.


