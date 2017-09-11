---
title: "Desproteção cargas cujas chaves foram revogados"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 6c4e6591-45d2-4d25-855e-062ad352d648
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/consumer-apis/dangerous-unprotect
ms.openlocfilehash: 44f21f380b994f46a8bb7368bca0cfc6e438ec4d
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/11/2017
---
# <a name="unprotecting-payloads-whose-keys-have-been-revoked"></a><span data-ttu-id="0d492-103">Desproteção cargas cujas chaves foram revogados</span><span class="sxs-lookup"><span data-stu-id="0d492-103">Unprotecting payloads whose keys have been revoked</span></span>

<a name=data-protection-consumer-apis-dangerous-unprotect></a>

<span data-ttu-id="0d492-104">A proteção de dados do ASP.NET Core APIs não são principalmente para persistência indefinida de cargas confidenciais.</span><span class="sxs-lookup"><span data-stu-id="0d492-104">The ASP.NET Core data protection APIs are not primarily intended for indefinite persistence of confidential payloads.</span></span> <span data-ttu-id="0d492-105">Outras tecnologias, como [Windows CNG DPAPI](https://msdn.microsoft.com/library/windows/desktop/hh706794%28v=vs.85%29.aspx) e [Azure Rights Management](https://technet.microsoft.com/library/jj585024.aspx) são mais adequados para o cenário de armazenamento indefinido, e eles têm recursos de gerenciamento de chave forte de forma correspondente.</span><span class="sxs-lookup"><span data-stu-id="0d492-105">Other technologies like [Windows CNG DPAPI](https://msdn.microsoft.com/library/windows/desktop/hh706794%28v=vs.85%29.aspx) and [Azure Rights Management](https://technet.microsoft.com/library/jj585024.aspx) are more suited to the scenario of indefinite storage, and they have correspondingly strong key management capabilities.</span></span> <span data-ttu-id="0d492-106">Dito isso, não há nada proibindo um desenvolvedor usa as APIs de proteção de dados ASP.NET Core para proteção de longo prazo de dados confidenciais.</span><span class="sxs-lookup"><span data-stu-id="0d492-106">That said, there is nothing prohibiting a developer from using the ASP.NET Core data protection APIs for long-term protection of confidential data.</span></span> <span data-ttu-id="0d492-107">Chaves nunca são removidas do anel de chave, portanto IDataProtector.Unprotect sempre pode recuperar cargas existentes, desde que as chaves estão disponíveis e válido.</span><span class="sxs-lookup"><span data-stu-id="0d492-107">Keys are never removed from the key ring, so IDataProtector.Unprotect can always recover existing payloads as long as the keys are available and valid.</span></span>

<span data-ttu-id="0d492-108">No entanto, um problema surge quando o desenvolvedor tenta Desproteger dados que foi protegidos com uma chave revogada, como IDataProtector.Unprotect lançará uma exceção nesse caso.</span><span class="sxs-lookup"><span data-stu-id="0d492-108">However, an issue arises when the developer tries to unprotect data that has been protected with a revoked key, as IDataProtector.Unprotect will throw an exception in this case.</span></span> <span data-ttu-id="0d492-109">Isso pode ser bom para cargas de curta duração ou transitórias (como tokens de autenticação), pois esses tipos de cargas podem ser facilmente recriados pelo sistema e na pior das hipóteses, o visitante do site pode ser necessário para fazer logon novamente.</span><span class="sxs-lookup"><span data-stu-id="0d492-109">This might be fine for short-lived or transient payloads (like authentication tokens), as these kinds of payloads can easily be recreated by the system, and at worst the site visitor might be required to log in again.</span></span> <span data-ttu-id="0d492-110">Mas cargas persistente, ter Desproteger throw pode levar a perda inaceitável de dados.</span><span class="sxs-lookup"><span data-stu-id="0d492-110">But for persisted payloads, having Unprotect throw could lead to unacceptable data loss.</span></span>

## <a name="ipersisteddataprotector"></a><span data-ttu-id="0d492-111">IPersistedDataProtector</span><span class="sxs-lookup"><span data-stu-id="0d492-111">IPersistedDataProtector</span></span>

<span data-ttu-id="0d492-112">Para suportar o cenário de permitir que conteúdos a ser desprotegido mesmo em face chaves revogadas, o sistema de proteção de dados contém um tipo de IPersistedDataProtector.</span><span class="sxs-lookup"><span data-stu-id="0d492-112">To support the scenario of allowing payloads to be unprotected even in the face of revoked keys, the data protection system contains an IPersistedDataProtector type.</span></span> <span data-ttu-id="0d492-113">Para obter uma instância de IPersistedDataProtector, obter uma instância de IDataProtector da maneira normal e tente converter IDataProtector para IPersistedDataProtector.</span><span class="sxs-lookup"><span data-stu-id="0d492-113">To get an instance of IPersistedDataProtector, simply get an instance of IDataProtector in the normal fashion and try casting the IDataProtector to IPersistedDataProtector.</span></span>

> [!NOTE]
> <span data-ttu-id="0d492-114">Nem todas as instâncias de IDataProtector podem ser convertidas em IPersistedDataProtector.</span><span class="sxs-lookup"><span data-stu-id="0d492-114">Not all IDataProtector instances can be cast to IPersistedDataProtector.</span></span> <span data-ttu-id="0d492-115">Os desenvolvedores devem usar o c# como operador ou semelhante evitar exceções de tempo de execução causado por conversões inválidos e devem estar preparados para tratar o caso de falha de maneira adequada.</span><span class="sxs-lookup"><span data-stu-id="0d492-115">Developers should use the C# as operator or similar to avoid runtime exceptions caused by invalid casts, and they should be prepared to handle the failure case appropriately.</span></span>

<span data-ttu-id="0d492-116">IPersistedDataProtector expõe a superfície de API a seguir:</span><span class="sxs-lookup"><span data-stu-id="0d492-116">IPersistedDataProtector exposes the following API surface:</span></span>

```csharp
DangerousUnprotect(byte[] protectedData, bool ignoreRevocationErrors,
     out bool requiresMigration, out bool wasRevoked) : byte[]
   ```

<span data-ttu-id="0d492-117">Essa API usa a carga protegida (como uma matriz de bytes) e retorna a carga desprotegida.</span><span class="sxs-lookup"><span data-stu-id="0d492-117">This API takes the protected payload (as a byte array) and returns the unprotected payload.</span></span> <span data-ttu-id="0d492-118">Não há nenhuma sobrecarga baseada em cadeia de caracteres.</span><span class="sxs-lookup"><span data-stu-id="0d492-118">There is no string-based overload.</span></span> <span data-ttu-id="0d492-119">Os dois parâmetros de saída são da seguinte maneira.</span><span class="sxs-lookup"><span data-stu-id="0d492-119">The two out parameters are as follows.</span></span>

* <span data-ttu-id="0d492-120">requiresMigration: será definido como true se a chave usada para proteger essa carga não é mais a chave padrão ativo, por exemplo, a chave usada para proteger essa carga é antiga e tem uma chave sem interrupção operação desde tomado local.</span><span class="sxs-lookup"><span data-stu-id="0d492-120">requiresMigration: will be set to true if the key used to protect this payload is no longer the active default key, e.g., the key used to protect this payload is old and a key rolling operation has since taken place.</span></span> <span data-ttu-id="0d492-121">O chamador poderá proteger novamente a carga dependendo de suas necessidades de negócios.</span><span class="sxs-lookup"><span data-stu-id="0d492-121">The caller may wish to consider reprotecting the payload depending on their business needs.</span></span>

* <span data-ttu-id="0d492-122">wasRevoked: será definido como verdadeiro se a chave usada para proteger essa carga foi revogada.</span><span class="sxs-lookup"><span data-stu-id="0d492-122">wasRevoked: will be set to true if the key used to protect this payload was revoked.</span></span>

>[!WARNING]
> <span data-ttu-id="0d492-123">Tenha bastante cuidado ao passar ignoreRevocationErrors: True para o método DangerousUnprotect.</span><span class="sxs-lookup"><span data-stu-id="0d492-123">Exercise extreme caution when passing ignoreRevocationErrors: true to the DangerousUnprotect method.</span></span> <span data-ttu-id="0d492-124">Se depois de chamar esse método, o valor de wasRevoked for true, em seguida, a chave usada para proteger essa carga foi revogada e autenticidade da carga deve ser tratada como suspeito.</span><span class="sxs-lookup"><span data-stu-id="0d492-124">If after calling this method the wasRevoked value is true, then the key used to protect this payload was revoked, and the payload's authenticity should be treated as suspect.</span></span> <span data-ttu-id="0d492-125">Nesse caso somente continuar a operar na carga de desprotegidos se você tiver alguma garantia separada que é autêntica, por exemplo, que ele do provenientes de um banco de dados seguro em vez de sendo enviada por um cliente da web não confiável.</span><span class="sxs-lookup"><span data-stu-id="0d492-125">In this case only continue operating on the unprotected payload if you have some separate assurance that it is authentic, e.g. that it's coming from a secure database rather than being sent by an untrusted web client.</span></span>

<span data-ttu-id="0d492-126">[!code-none[Main](dangerous-unprotect/samples/dangerous-unprotect.cs)]</span><span class="sxs-lookup"><span data-stu-id="0d492-126">[!code-none[Main](dangerous-unprotect/samples/dangerous-unprotect.cs)]</span></span>