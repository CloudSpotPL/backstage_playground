# Hello Docs
---
## HL Azure China Subscription Onboarding process

```kroki-mermaid
sequenceDiagram
autonumber
    actor Requestor
    actor China Regulatory Council
    
    China Regulatory Council->>Vending Machine: Provide Requestor CDSID, AppID, ArchType, vNet size for Corp
    Vending Machine->>China Regulatory Council: Notification: Subscriptions with AppId XXXX was just deployed by CDSID@volvocars.com
    Vending Machine->>Requestor: Notification: Subscriptions with AppId XXXX was just deployed for you
```

# HL Architecture
## Components
- [GH repository](https://github.com/volvo-cars/mb-azure-china_vending_machine/edit/main/README.md)
- [GH SH Runner - pce-gh-runner-vem-001](https://portal.azure.com/#@volvocars.onmicrosoft.com/resource/subscriptions/0c6364d5-fd1c-4270-a5dd-bfe600e46d04/resourceGroups/rg-vending-machine-internal-gh-runner/providers/Microsoft.Compute/virtualMachines/gh-runner-001/overview)
- [GH SH Runner - pce-gh-runner-vem-002](https://portal.azure.com/#@volvocars.onmicrosoft.com/resource/subscriptions/0c6364d5-fd1c-4270-a5dd-bfe600e46d04/resourceGroups/rg-vending-machine-internal-gh-runner/providers/Microsoft.Compute/virtualMachines/gh-runner-001/overview)
- [MSForms](https://forms.office.com/e/gq2B8MmpTR) 
- [LogicApps-VendingMachineFeeder](https://portal.azure.com/#@volvocars.onmicrosoft.com/resource/subscriptions/d563d8cd-37a4-4ef8-a495-cc6347ba8124/resourceGroups/rg-cn-vending_machine/providers/Microsoft.Logic/workflows/VendingMachineFeeder/logicApp)
- [StorageAccount-SubscriptionsInventory](https://portal.azure.com/#@volvocars.onmicrosoft.com/resource/subscriptions/d563d8cd-37a4-4ef8-a495-cc6347ba8124/resourceGroups/rg-cn-vending_machine/providers/Microsoft.Storage/storageAccounts/stcnvendingmachine001/overview)
- [VendingMachineNotifier](https://portal.azure.com/#@volvocars.onmicrosoft.com/resource/subscriptions/d563d8cd-37a4-4ef8-a495-cc6347ba8124/resourceGroups/rg-cn-vending_machine/providers/Microsoft.Logic/workflows/VendingMachineNotifier/logicApp)

# HL Vending Machine flow

```kroki-mermaid
sequenceDiagram

    actor EA Account Owner
    EA Account Owner-->> EA Portal: Create Azure CN subscriptions
    EA Account Owner-->> Subscriptions Inventory: Add created subscriptions to the inventory
    autonumber
    actor Requestor
    box Gray Vending Machine
    participant Requestor
    participant Order Form
    participant Vending Machine Feeder
    participant Vending Machine
    participant MSGraph
    participant MS Teams - PCE
    participant email - Requestor
    end
    Requestor->>Order Form: (AppID, vNET_size, ArchType)
    Order Form->> Vending Machine Feeder: Feeder Payload
    Vending Machine Feeder->>Subscriptions Inventory: Get free subscriptions
    Vending Machine Feeder-->>EA Account Owner: Send notification if there are no free subscriptions
    Vending Machine Feeder->>IPAM: Get free CIDR
    Vending Machine Feeder-->>NetOps: Send notification if there are no free CIDRs
    Vending Machine Feeder->>MSGraph: Get requestor profile
    Vending Machine Feeder->> Vending Machine: Payload
    Vending Machine-->>MS Teams - PCE: Send notification if subscriptions createion failed
    Vending Machine-->>email - Requestor: Send Requestor notification
```

## Vending Machine LL flow
```mermaid
flowchart TD
    X[Vending Machine Feeder]-->A[Payload Non-Prod]
    A-->AA[Create Subscription]
    AA-->AAA[Apply Tags]
    AAA-->C[Create Subscription Owners Security Group]
    C-->D[Assign Reader role to the subscription]
    D-->E[Create Sub Owners SPN]
    E-->F[Assign OwnerLimit-Action to the SPN]
    F-->G[Create Network Resource Group]
    G-->H[Create NSG]
    H-->I[Create vNET]
    I-->J[Create subnet]
    J-->K[Assign NSG to the Subnet]
    K-->L[Network peering]
    L-->M[Route Table and assignment]
    M-->N[Create HUB route]
    N-->O[Move Subscription to the proper Arch Type Management Group]
    O-->P[Configure default NSG and Routing Table]
    
    X[Vending Machine Feeder]-->A1[Payload Prod]
    A1-->AA1[Create Subscription]
    AA1-->AAA1[Apply Tags]
    AAA1-->C1[Create Subscription Owners Security Group]
    C1-->D1[Assign Reader role to the subscription]
    D1-->E1[Create Sub Owners SPN]
    E1-->F1[Assign OwnerLimit-Action to the SPN]
    F1-->G1[Create Network Resource Group]
    G1-->H1[Create NSG]
    H1-->I1[Create vNET]
    I1-->J1[Create subnet]
    J1-->K1[Assign NSG to the Subnet]
    K1-->L1[Network peering]
    L1-->M1[Route Table and assignment]
    M1-->N1[Create HUB route]
    N1-->O1[Move Subscription to the proper Arch Type Management Group]
    O1-->P1[Configure default NSG and Routing Table]

```
## Vending Machine artifacts naming convention
|Item|Naming convention|Comment|
|--|--|--|
| ${var.app_id}-${var.arch_type}-${lower(var.env_type)}-${var.suffix} | |
| rg-network-${lower(var.env_type)}-001 | |
| AAD Security Group | cld-cn-${var.app_id}-${var.arch_type}-${var.env_type}-${var.suffix}-sg | |
| AAD SPN | spn-cn-${var.app_id}-${var.arch_type}-${var.env_type}-${var.suffix} | |
| Network Security Group | nsg-${var.short_location}-${var.app_id}-${lower(var.env_type)}-001 | |
| Virtual Network | vnet-${var.short_location}-${var.app_id}-${lower(var.env_type)}-001 | |
| Subnet | snet-${var.short_location}-${var.app_id}-${lower(var.env_type)}-001 | |
| vNET Peering | peer-${var.short_location}-${var.app_id}-${lower(var.env_type)}-${var.suffix} | |
| Route Table | rt-${var.short_location}-${var.app_id}-${lower(var.env_type)}-001 | |
| Route | ro-vnet-${var.short_location}-${var.app_id}-${lower(var.env_type)}-${var.suffix} | | 
| Policy Assignment | [VCC]ConfigureDefaultNSGandRTonSub | |

where: \
var.short_location - cn3 \
var.app_id - Application ID \
var.arch_type - Architecture Type (Online|Corp) \
var.env_type - Environment (prod|nonprod) \
var.suffix - Random 3 digit suffix, subscription specific


