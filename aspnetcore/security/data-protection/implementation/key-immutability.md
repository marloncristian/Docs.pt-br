---
title: "A imutabilidade de chave e alterar as configurações"
author: rick-anderson
description: "Este documento descreve os detalhes de implementação de ASP.NET Core dados proteção chave imutabilidade APIs."
keywords: "ASP.NET Core, proteção de dados, imutabilidade de chave"
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: fc911ae3-feca-4ffe-8b43-362bc632fe04
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/implementation/key-immutability
ms.openlocfilehash: 96860b44b64f241a1bbff2ac8366e0863b1ac10c
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/10/2017
---
# <a name="key-immutability-and-changing-settings"></a>A imutabilidade de chave e alterar as configurações

Depois que um objeto é persistido para o armazenamento de backup, sua representação para sempre é fixo. Novos dados podem ser adicionados ao repositório de backup, mas os dados existentes nunca podem ser modificados. O objetivo principal deste comportamento é evitar a corrupção de dados.

Uma consequência esse comportamento é que, quando uma chave é gravada para armazenamento de backup, é imutável. As datas de criação, ativação e expiração nunca podem ser alteradas, embora ele pode ser revogada usando `IKeyManager`. Além disso, suas informações algorítmicos subjacentes, material de chave mestre e criptografia propriedades rest também são imutáveis.

Se o desenvolvedor altera qualquer configuração que afeta a persistência de chave, essas alterações não terão efeito até a próxima vez que uma chave é gerada, através de uma chamada explícita para `IKeyManager.CreateNewKey` ou por meio do sistema proteção de dados próprio [chave automática geração](key-management.md#data-protection-implementation-key-management) comportamento. As configurações que afetam a persistência de chave são da seguinte maneira:

* [O tempo de vida de chave padrão](key-management.md#data-protection-implementation-key-management)

* [A criptografia de chave no mecanismo de rest](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest)

* [As informações de algoritmos contidas na chave do](xref:security/data-protection/configuration/overview#changing-algorithms-with-usecryptographicalgorithms)

Se você precisar essas configurações de anteriores a próxima chave automática sem interrupção tempo, considere a possibilidade de fazer uma chamada explícita para `IKeyManager.CreateNewKey` para forçar a criação de uma nova chave. Lembre-se de fornecer uma data de ativação explícita ({agora + 2 dias} é uma boa regra prática para permitir um tempo para que a alteração se propague) e a data de expiração na chamada.

>[!TIP]
> Tocar o repositório de todos os aplicativos devem especificar as mesmas configurações com o `IDataProtectionBuilder` métodos de extensão. Caso contrário, as propriedades da chave persistente dependerá do aplicativo específico que invocou as rotinas de geração de chave.
