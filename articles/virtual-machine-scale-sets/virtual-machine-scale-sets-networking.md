---
title: Azure 仮想マシン スケール セットのネットワーク | Microsoft Docs
description: Azure 仮想マシン スケール セットのネットワーク プロパティの構成。
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 76ac7fd7-2e05-4762-88ca-3b499e87906e
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/17/2017
ms.author: negat
ms.openlocfilehash: abad57856db63c954f963a28b1dbd3c95395c9bd
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/01/2018
ms.locfileid: "34652588"
---
# <a name="networking-for-azure-virtual-machine-scale-sets"></a>Azure 仮想マシン スケール セットのネットワーク

ポータルを通じて Azure 仮想マシン スケール セットをデプロイすると、特定のネットワーク プロパティには既定値が設定されます。たとえば、受信 NAT 規則付きの Azure ロード バランサーです。 この記事では、スケール セットで構成できる、いくつかのより高度なネットワーク機能の使用方法について説明します。

この記事で説明されているすべての機能は、Azure Resource Manager テンプレートを使用して構成することができます。 一部の機能については、Azure CLI と PowerShell の例も含まれています。 CLI 2.10、および PowerShell 4.2.0 以降を使用してください。

## <a name="accelerated-networking"></a>高速ネットワーク
Azure 高速ネットワークでは、仮想マシンでシングルルート I/O 仮想化 (SR-IOV) が可能になることにより、ネットワークのパフォーマンスが向上します。 高速ネットワークの使用方法について詳しくは、[Windows](../virtual-network/create-vm-accelerated-networking-powershell.md) または [Linux](../virtual-network/create-vm-accelerated-networking-cli.md) 仮想マシンの高速ネットワークに関する記事をご覧ください。 スケール セットで高速ネットワークを使用するには、スケール セットの networkInterfaceConfigurations 設定で enableAcceleratedNetworking を **true** に設定します。 例: 
```json
"networkProfile": {
    "networkInterfaceConfigurations": [
    {
      "name": "niconfig1",
      "properties": {
        "primary": true,
        "enableAcceleratedNetworking" : true,
        "ipConfigurations": [
          ...
        ]
      }
    }
   ]
}
```

## <a name="create-a-scale-set-that-references-an-existing-azure-load-balancer"></a>既存の Azure ロード バランサーを参照するスケール セットの作成
Azure Portal を使用してスケール セットを作成した場合、ほとんどの構成オプションで新しいロード バランサーが作成されます。 既存のロード バランサーを参照する必要があるスケール セットを作成する場合は、CLI を使用して作成することができます。 次のサンプル スクリプトは、ロード バランサーを作成してから、それを参照するスケール セットを作成します。
```bash
az network lb create -g lbtest -n mylb --vnet-name myvnet --subnet mysubnet --public-ip-address-allocation Static --backend-pool-name mybackendpool

az vmss create -g lbtest -n myvmss --image Canonical:UbuntuServer:16.04-LTS:latest --admin-username negat --ssh-key-value /home/myuser/.ssh/id_rsa.pub --upgrade-policy-mode Automatic --instance-count 3 --vnet-name myvnet --subnet mysubnet --lb mylb --backend-pool-name mybackendpool

```

## <a name="create-a-scale-set-that-references-an-application-gateway"></a>Application Gateway を参照するスケール セットを作成する
アプリケーション ゲートウェイを使うスケール セットを作成するには、次の ARM テンプレート構成のように、スケール セットの ipConfigurations セクションにおいてアプリケーション ゲートウェイのバックエンド アドレス プールを参照します。
```json
"ipConfigurations": [{
  "name": "{config-name}",
  "properties": {
  "subnet": {
    "id": "{subnet-id}"
  },
  "ApplicationGatewayBackendAddressPools": [{
    "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/applicationGateways/{gateway-name}/backendAddressPools/{pool-name}"
  }]
}]
```

>[!NOTE]
> アプリケーション ゲートウェイは、スケール セットと同じ仮想ネットワーク内の、スケール セットとは異なるサブネット内に存在する必要があることに、注意してください。


## <a name="configurable-dns-settings"></a>構成可能な DNS 設定
スケール セットは、それが作成された VNET とサブネットの特定の DNS 設定を既定で引き継ぎます。 しかし、スケール セットの DNS 設定は直接構成することができます。

### <a name="creating-a-scale-set-with-configurable-dns-servers"></a>構成可能な DNS サーバーによるスケール セットの作成
CLI 2.0 を使用してカスタム DNS 構成でスケール セットを作成するには、**--dns-servers** 引数とスペース区切りのサーバー IP アドレスを **vmss create** コマンドに追加します。 例: 
```bash
--dns-servers 10.0.0.6 10.0.0.5
```
Azure テンプレート内でカスタム DNS サーバーを構成するには、スケール セットの networkInterfaceConfigurations セクションに dnsSettings プロパティを追加します。 例: 
```json
"dnsSettings":{
    "dnsServers":["10.0.0.6", "10.0.0.5"]
}
```

### <a name="creating-a-scale-set-with-configurable-virtual-machine-domain-names"></a>構成可能な仮想マシン ドメイン名を備えたスケール セットの作成
CLI 2.0 を使用して仮想マシンのカスタム DNS 名を備えたスケール セットを作成するには、**--vm-domain-name** 引数とドメイン名を表す文字列を **vmss create** コマンドに追加します。

Azure テンプレート内でドメイン名を設定するには、スケール セットの **networkInterfaceConfigurations** セクションに **dnsSettings** プロパティを追加します。 例: 

```json
"networkProfile": {
  "networkInterfaceConfigurations": [
    {
    "name": "nic1",
    "properties": {
      "primary": "true",
      "ipConfigurations": [
      {
        "name": "ip1",
        "properties": {
          "subnet": {
            "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', variables('vnetName'), '/subnets/subnet1')]"
          },
          "publicIPAddressconfiguration": {
            "name": "publicip",
            "properties": {
            "idleTimeoutInMinutes": 10,
              "dnsSettings": {
                "domainNameLabel": "[parameters('vmssDnsName')]"
              }
            }
          }
        }
      }
    ]
    }
}
```

個々の仮想マシンの DNS 名に対する出力は、次の形式になります。 
```
<vm><vmindex>.<specifiedVmssDomainNameLabel>
```

## <a name="public-ipv4-per-virtual-machine"></a>仮想マシンごとのパブリック IPv4
一般に、Azure スケール セット仮想マシンには、独自のパブリック IP アドレスは不要です。 ほとんどのシナリオでは、パブリック IP アドレスをロード バランサーまたは個々の仮想マシン (別名、ジャンプボックス) に関連付ける方が、経済的で安全です。そうすれば、受信接続は必要に応じて (たとえば、受信 NAT 規則を通じて) スケール セット仮想マシンにルーティングされます。

ただし、一部のシナリオでは、スケール セット仮想マシンが独自のパブリック IP アドレスを備える必要があります。 たとえば、ゲームで、ゲームの物理処理を行っているクラウド仮想マシンにコンソールが直接接続する必要がある場合です。 もう 1 つの例は、分散型データベースで複数のリージョンにわたって各仮想マシンが互いに対する外部接続を確立する必要がある場合です。

### <a name="creating-a-scale-set-with-public-ip-per-virtual-machine"></a>仮想マシンごとにパブリック IP アドレスがあるスケール セットの作成
CLI 2.0 を使用して、各仮想マシンにパブリック IP アドレスが割り当てられるスケール セットを作成するには、**vmss create** コマンドに **--public-ip-per-vm** パラメーターを追加します。 

Azure テンプレートを使用してスケール セットを作成するには、Microsoft.Compute/virtualMachineScaleSets リソースの API バージョンが少なくとも **2017-03-30** であることを確認し、スケール セットの ipConfigurations セクションに **publicIpAddressConfiguration** JSON プロパティを追加します。 例: 

```json
"publicIpAddressConfiguration": {
    "name": "pub1",
    "properties": {
      "idleTimeoutInMinutes": 15
    }
}
```
サンプル テンプレート: [201-vmss-public-ip-linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-public-ip-linux)

### <a name="querying-the-public-ip-addresses-of-the-virtual-machines-in-a-scale-set"></a>スケール セットに含まれた仮想マシンのパブリック IP アドレスの照会
CLI 2.0 を使用して、スケール セット仮想マシンに割り当てられているパブリック IP アドレスの一覧を表示するには、**az vmss list-instance-public-ips** コマンドを使用します。

PowerShell を使用して、スケール セットのパブリック IP アドレスを一覧表示するには、_Get-AzureRmPublicIpAddress_ コマンドを使用します。 例: 
```PowerShell
PS C:\> Get-AzureRmPublicIpAddress -ResourceGroupName myrg -VirtualMachineScaleSetName myvmss
```

また、パブリック IP アドレス構成のリソース ID を直接参照して、パブリック IP アドレスを照会することもできます。 例: 
```PowerShell
PS C:\> Get-AzureRmPublicIpAddress -ResourceGroupName myrg -Name myvmsspip
```

スケール セット仮想マシンに割り当てられているパブリック IP アドレスを照会するには、[Azure Resource Explorer](https://resources.azure.com) または Azure REST API のバージョン **2017-03-30** 以降を使用します。

Resource Explorer を使用してスケール セットのパブリック IP アドレスを表示するには、スケール セットの **publicipaddresses** セクションを見ます。 例: https://resources.azure.com/subscriptions/_your_sub_id_/resourceGroups/_your_rg_/providers/Microsoft.Compute/virtualMachineScaleSets/_your_vmss_/publicipaddresses

```
GET https://management.azure.com/subscriptions/{your sub ID}/resourceGroups/{RG name}/providers/Microsoft.Compute/virtualMachineScaleSets/{scale set name}/publicipaddresses?api-version=2017-03-30
```

出力例:
```json
{
  "value": [
    {
      "name": "pub1",
      "id": "/subscriptions/your-subscription-id/resourceGroups/your-rg/providers/Microsoft.Compute/virtualMachineScaleSets/pipvmss/virtualMachines/0/networkInterfaces/pipvmssnic/ipConfigurations/yourvmssipconfig/publicIPAddresses/pub1",
      "etag": "W/\"a64060d5-4dea-4379-a11d-b23cd49a3c8d\"",
      "properties": {
        "provisioningState": "Succeeded",
        "resourceGuid": "ee8cb20f-af8e-4cd6-892f-441ae2bf701f",
        "ipAddress": "13.84.190.11",
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Dynamic",
        "idleTimeoutInMinutes": 15,
        "ipConfiguration": {
          "id": "/subscriptions/your-subscription-id/resourceGroups/your-rg/providers/Microsoft.Compute/virtualMachineScaleSets/yourvmss/virtualMachines/0/networkInterfaces/yourvmssnic/ipConfigurations/yourvmssipconfig"
        }
      }
    },
    {
      "name": "pub1",
      "id": "/subscriptions/your-subscription-id/resourceGroups/your-rg/providers/Microsoft.Compute/virtualMachineScaleSets/yourvmss/virtualMachines/3/networkInterfaces/yourvmssnic/ipConfigurations/yourvmssipconfig/publicIPAddresses/pub1",
      "etag": "W/\"5f6ff30c-a24c-4818-883c-61ebd5f9eee8\"",
      "properties": {
        "provisioningState": "Succeeded",
        "resourceGuid": "036ce266-403f-41bd-8578-d446d7397c2f",
        "ipAddress": "13.84.159.176",
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Dynamic",
        "idleTimeoutInMinutes": 15,
        "ipConfiguration": {
          "id": "/subscriptions/your-subscription-id/resourceGroups/your-rg/providers/Microsoft.Compute/virtualMachineScaleSets/yourvmss/virtualMachines/3/networkInterfaces/yourvmssnic/ipConfigurations/yourvmssipconfig"
        }
      }
    }
```

## <a name="multiple-ip-addresses-per-nic"></a>NIC ごとの複数の IP アドレス
スケール セット内の VM に接続されたすべての NIC には、1 つ以上の IP 構成を関連付けることができます。 各構成には、1 つのプライベート IP アドレスが割り当てられます。 また、1 つのパブリック IP アドレス リソースが関連付けられる場合もあります。 いくつの IP アドレスを NIC に割り当てることができるかと、Azure サブスクリプションでいくつのパブリック IP アドレスを使用できるかを理解するには、[Azure の制限](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits)に関するページを参照してください。

## <a name="multiple-nics-per-virtual-machine"></a>仮想マシンごとの複数の NIC
マシンのサイズに応じて、仮想マシンごとに最大 8 個の NIC を使用することができます。 マシンごとの NIC の最大数については、[VM サイズの記事](../virtual-machines/windows/sizes.md)を参照してください。 VM インスタンスに接続されているすべての NIC は、同じ仮想ネットワークに接続する必要があります。 NIC は異なるサブネットに接続できますが、すべてのサブネットは同じ仮想ネットワークに属している必要があります。

次の例のスケール セット ネットワーク プロファイルは、複数の NIC エントリと、仮想マシンごとの複数のパブリック IP を示しています。

```json
"networkProfile": {
    "networkInterfaceConfigurations": [
        {
        "name": "nic1",
        "properties": {
            "primary": "true",
            "ipConfigurations": [
            {
                "name": "ip1",
                "properties": {
                "subnet": {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', variables('vnetName'), '/subnets/subnet1')]"
                },
                "publicipaddressconfiguration": {
                    "name": "pub1",
                    "properties": {
                    "idleTimeoutInMinutes": 15
                    }
                },
                "loadBalancerInboundNatPools": [
                    {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/inboundNatPools/natPool1')]"
                    }
                ],
                "loadBalancerBackendAddressPools": [
                    {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/backendAddressPools/addressPool1')]"
                    }
                ]
                }
            }
            ]
        }
        },
        {
        "name": "nic2",
        "properties": {
            "primary": "false",
            "ipConfigurations": [
            {
                "name": "ip1",
                "properties": {
                "subnet": {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', variables('vnetName'), '/subnets/subnet1')]"
                },
                "publicipaddressconfiguration": {
                    "name": "pub1",
                    "properties": {
                    "idleTimeoutInMinutes": 15
                    }
                },
                "loadBalancerInboundNatPools": [
                    {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/inboundNatPools/natPool1')]"
                    }
                ],
                "loadBalancerBackendAddressPools": [
                    {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/backendAddressPools/addressPool1')]"
                    }
                ]
                }
            }
            ]
        }
        }
    ]
}
```

## <a name="nsg-per-scale-set"></a>スケール セットごとの NSG
ネットワーク セキュリティ グループは、スケール セットに直接適用できます。そのためには、スケール セット仮想マシン プロパティのネットワーク インターフェイス構成セクションに参照を追加します。

例:  
```
"networkProfile": {
    "networkInterfaceConfigurations": [
        {
            "name": "nic1",
            "properties": {
                "primary": "true",
                "ipConfigurations": [
                    {
                        "name": "ip1",
                        "properties": {
                            "subnet": {
                                "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', variables('vnetName'), '/subnets/subnet1')]"
                            }
                "loadBalancerInboundNatPools": [
                                {
                                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/inboundNatPools/natPool1')]"
                                }
                            ],
                            "loadBalancerBackendAddressPools": [
                                {
                                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/backendAddressPools/addressPool1')]"
                                }
                            ]
                        }
                    }
                ],
                "networkSecurityGroup": {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', variables('nsgName'))]"
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>次の手順
Azure 仮想ネットワークの詳細については、「[Azure Virtual Network の概要](../virtual-network/virtual-networks-overview.md)」を参照してください。
