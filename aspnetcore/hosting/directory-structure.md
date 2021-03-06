---
title: "Estrutura de diretórios do ASP.NET Core"
author: guardrex
description: "A estrutura de diretório de aplicativos publicados do ASP.NET Core."
keywords: "ASP.NET Core, a estrutura de diretórios"
ms.author: riande
manager: wpickett
ms.date: 03/15/2017
ms.topic: article
ms.assetid: e55eb131-d42e-4bf6-b130-fd626082243c
ms.technology: aspnet
ms.prod: asp.net-core
uid: hosting/directory-structure
ms.openlocfilehash: 60797bff85a44dd10caad4aabc109ee12dedfe61
ms.sourcegitcommit: 7efdc4b6025ad70c15c26bf7451c3c0411123794
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/02/2017
---
# <a name="directory-structure-of-published-aspnet-core-apps"></a>Estrutura de diretório de aplicativos do ASP.NET Core publicados

Por [Luke Latham](https://github.com/guardrex)

No núcleo do ASP.NET, o diretório de aplicativo, *publicar*, é composta de arquivos de aplicativo, os arquivos de configuração, ativos estáticos, pacotes e o tempo de execução (para aplicativos independentes). Essa é a mesma estrutura de diretório que versões anteriores do ASP.NET, onde o aplicativo inteiro reside no diretório raiz da web.

| Tipo de aplicativo | Estrutura de diretórios |
| --- | --- |
| Dependente de estrutura de implantação | <ul><li>Publicar\*<ul><li>logs de\* (se for incluído em publishOptions)</li><li>refs\*</li><li>tempos de execução\*</li><li>Modos de exibição\* (se for incluído em publishOptions)</li><li>wwwroot\* (se for incluído em publishOptions)</li><li>Arquivos .dll</li><li>MyApp.DEPs.JSON</li><li>como Myapp.dll</li><li>MyApp.PDB</li><li>MyApp. PrecompiledViews.dll (se pré-compilando exibições Razor)</li><li>MyApp. PrecompiledViews.pdb (se pré-compilando exibições Razor)</li><li>MyApp.runtimeconfig.JSON</li><li>Web. config (se for incluído em publishOptions)</li></ul></li></ul> |
| Independente de implantação | <ul><li>Publicar\*<ul><li>logs de\* (se for incluído em publishOptions)</li><li>refs\*</li><li>Modos de exibição\* (se for incluído em publishOptions)</li><li>wwwroot\* (se for incluído em publishOptions)</li><li>Arquivos .dll</li><li>MyApp.DEPs.JSON</li><li>MyApp.exe</li><li>MyApp.PDB</li><li>MyApp. PrecompiledViews.dll (se pré-compilando exibições Razor)</li><li>MyApp. PrecompiledViews.pdb (se pré-compilando exibições Razor)</li><li>MyApp.runtimeconfig.JSON</li><li>Web. config (se for incluído em publishOptions)</li></ul></li></ul> |
\*Indica um diretório

O conteúdo do *publicar* diretório representa o *caminho raiz de conteúdo*, também chamado de *caminho base do aplicativo*, da implantação. O nome que é fornecido para o *publicar* serve de seu local de diretório na implantação, como a caminho físico do servidor para o aplicativo hospedado. O *wwwroot* diretório, se presente, contém ativos estáticos somente. O *logs* diretório pode ser incluído na implantação ao criá-lo no projeto e adicionar o `<Target>` elemento mostrado abaixo ao seu *. csproj* arquivo ou criando o diretório fisicamente no servidor.

```xml
<Target Name="CreateLogsFolder" AfterTargets="Publish">
  <MakeDir Directories="$(PublishDir)Logs" 
           Condition="!Exists('$(PublishDir)Logs')" />
  <WriteLinesToFile File="$(PublishDir)Logs\.log" 
                    Lines="Generated file" 
                    Overwrite="True" 
                    Condition="!Exists('$(PublishDir)Logs\.log')" />
</Target>
```

O `<MakeDir>` elemento cria vazio *Logs* pasta na saída publicada. O elemento usa o `PublishDir` propriedade para determinar o local de destino para criar a pasta. Vários métodos de implantação, como a implantação da Web, ignore as pastas vazias durante a implantação. O `<WriteLinesToFile>` elemento gera um arquivo no *Logs* pasta, o que garante a implantação da pasta para o servidor. Observe que a criação de pasta ainda pode falhar se o processo do operador não tem acesso de gravação para a pasta de destino.

O diretório de implantação requer permissões de leitura/execução, enquanto o *Logs* directory requer permissões de leitura/gravação. Diretórios adicionais onde ativos serão gravados exigem permissões de leitura/gravação.
