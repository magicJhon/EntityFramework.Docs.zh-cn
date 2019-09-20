---
title: Azure Cosmos DB 提供程序-使用非结构化数据-EF Core
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 09/12/2019
ms.assetid: b47d41b6-984f-419a-ab10-2ed3b95e3919
uid: core/providers/cosmos/unstructured-data
ms.openlocfilehash: 86bb0f7915c8a2561e7d5cd5dffc27474218a112
ms.sourcegitcommit: cbaa6cc89bd71d5e0bcc891e55743f0e8ea3393b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/20/2019
ms.locfileid: "71150769"
---
# <a name="working-with-unstructured-data-in-ef-core-azure-cosmos-db-provider"></a><span data-ttu-id="908e9-102">使用 EF Core Azure Cosmos DB 提供程序中的非结构化数据</span><span class="sxs-lookup"><span data-stu-id="908e9-102">Working with Unstructured Data in EF Core Azure Cosmos DB Provider</span></span>

<span data-ttu-id="908e9-103">EF Core 旨在使使用在模型中定义的架构的数据变得简单。</span><span class="sxs-lookup"><span data-stu-id="908e9-103">EF Core was designed to make it easy to work with data that follows a schema defined in the model.</span></span> <span data-ttu-id="908e9-104">但 Azure Cosmos DB 的优点之一是存储数据形状的灵活性。</span><span class="sxs-lookup"><span data-stu-id="908e9-104">However one of the strengths of Azure Cosmos DB is the flexibility in the shape of the data stored.</span></span>

## <a name="accessing-the-raw-json"></a><span data-ttu-id="908e9-105">访问原始 JSON</span><span class="sxs-lookup"><span data-stu-id="908e9-105">Accessing the raw JSON</span></span>

<span data-ttu-id="908e9-106">可以通过名为`"__jObject"`的[卷影状态](../../modeling/shadow-properties.md)中的一个特殊属性来访问未跟踪的属性，该属性包含`JObject`表示从存储中接收的数据的 EF Core 和将存储的数据：</span><span class="sxs-lookup"><span data-stu-id="908e9-106">It is possible to access the properties that are not tracked by EF Core through a special property in [shadow-state](../../modeling/shadow-properties.md) named `"__jObject"` that contains a `JObject` representing the data recieved from the store and data that will be stored:</span></span>

[!code-csharp[Unmapped](../../../../samples/core/Cosmos/UnstructuredData/Sample.cs?highlight=21-23&name=Unmapped)]

``` json
{
    "Id": 1,
    "Discriminator": "Order",
    "TrackingNumber": null,
    "id": "Order|1",
    "Address": {
        "ShipsToCity": "London",
        "Discriminator": "StreetAddress",
        "ShipsToStreet": "221 B Baker St"
    },
    "_rid": "eLMaAK8TzkIBAAAAAAAAAA==",
    "_self": "dbs/eLMaAA==/colls/eLMaAK8TzkI=/docs/eLMaAK8TzkIBAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-683e-0a12bf8d01d5\"",
    "_attachments": "attachments/",
    "BillingAddress": "Clarence House",
    "_ts": 1568164374
}
```

> [!WARNING]
> <span data-ttu-id="908e9-107">该`"__jObject"`属性是 EF Core 基础结构的一部分，只应用作最后的手段，因为在将来的版本中可能会有不同的行为。</span><span class="sxs-lookup"><span data-stu-id="908e9-107">The `"__jObject"` property is part of the EF Core infrastructure and should only be used as a last resort as it is likely to have different behavior in future releases.</span></span>

> [!NOTE]
> <span data-ttu-id="908e9-108">对实体所做的更改将覆盖在`"__jObject"`过程`SaveChanges`中存储的值。</span><span class="sxs-lookup"><span data-stu-id="908e9-108">Changes to the entity will override the values stored in `"__jObject"` during `SaveChanges`.</span></span>

## <a name="using-cosmosclient"></a><span data-ttu-id="908e9-109">使用 CosmosClient</span><span class="sxs-lookup"><span data-stu-id="908e9-109">Using CosmosClient</span></span>

<span data-ttu-id="908e9-110">若要完全分离 EF Core 从获取`CosmosClient` `DbContext` [Azure Cosmos DB SDK 的一部分](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started)的对象：</span><span class="sxs-lookup"><span data-stu-id="908e9-110">To decouple completely from EF Core get the `CosmosClient` object that is [part of the Azure Cosmos DB SDK](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started) from `DbContext`:</span></span>

[!code-csharp[CosmosClient](../../../../samples/core/Cosmos/UnstructuredData/Sample.cs?highlight=3&name=CosmosClient)]

## <a name="missing-property-values"></a><span data-ttu-id="908e9-111">缺少属性值</span><span class="sxs-lookup"><span data-stu-id="908e9-111">Missing property values</span></span>

<span data-ttu-id="908e9-112">在上面的示例中，我们`"TrackingNumber"`从顺序中删除了属性。</span><span class="sxs-lookup"><span data-stu-id="908e9-112">In the previous example we removed the `"TrackingNumber"` property from the order.</span></span> <span data-ttu-id="908e9-113">由于在 Cosmos DB 中索引的工作原理，引用缺少的属性的其他位置的查询可能会返回意外的结果。</span><span class="sxs-lookup"><span data-stu-id="908e9-113">Because of how indexing works in Cosmos DB, queries that reference the missing property somewhere else than in the projection could return unexpected results.</span></span> <span data-ttu-id="908e9-114">例如:</span><span class="sxs-lookup"><span data-stu-id="908e9-114">For example:</span></span>

[!code-csharp[MissingProperties](../../../../samples/core/Cosmos/UnstructuredData/Sample.cs?name=MissingProperties)]

<span data-ttu-id="908e9-115">排序的查询实际上不返回任何结果。</span><span class="sxs-lookup"><span data-stu-id="908e9-115">The sorted query actually returns no results.</span></span> <span data-ttu-id="908e9-116">这意味着，当直接使用存储时，应注意始终填充由 EF Core 映射的属性。</span><span class="sxs-lookup"><span data-stu-id="908e9-116">This means that one should take care to always populate properties mapped by EF Core when working with the store directly.</span></span>

> [!NOTE]
> <span data-ttu-id="908e9-117">在未来版本的 Cosmos 中，此行为可能会发生变化。</span><span class="sxs-lookup"><span data-stu-id="908e9-117">This behavior might change in future versions of Cosmos.</span></span> <span data-ttu-id="908e9-118">例如，当前如果索引策略定义了复合索引 {Id/？</span><span class="sxs-lookup"><span data-stu-id="908e9-118">For instance, currently if the indexing policy defines the composite index {Id/?</span></span> <span data-ttu-id="908e9-119">ASC，TrackingNumber/？</span><span class="sxs-lookup"><span data-stu-id="908e9-119">ASC, TrackingNumber/?</span></span> <span data-ttu-id="908e9-120">ASC）}，then "ORDER BY c.Id asc，.c asc" 的查询__将__返回缺少属性的`"TrackingNumber"`项。</span><span class="sxs-lookup"><span data-stu-id="908e9-120">ASC)}, then a query the has 'ORDER BY c.Id ASC, c.Discriminator ASC' __would__ return items that are missing the `"TrackingNumber"` property.</span></span>