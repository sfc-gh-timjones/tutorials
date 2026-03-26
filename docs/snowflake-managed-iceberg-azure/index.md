# Snowflake Managed Iceberg Tables (Azure)

Configure Snowflake-managed Apache Iceberg tables using Azure ADLS Gen2 as external storage.

!!! warning "Adapt to Your Environment"
    This guide is a reference implementation demonstrated in a simplified environment.
    Your networking, security policies, IAM configurations, and infrastructure will differ.
    Always validate against your organization's requirements before implementing.

---

## 1. Provision Cloud Storage (Azure ADLS)

Create an Azure Storage Account with ADLS Gen2 to store Snowflake-managed Iceberg tables.

!!! abstract "What's happening"
    At this stage, you are creating the cloud storage layer where Snowflake will write Iceberg data files and metadata. Think of this as provisioning the "landing zone" for your open lakehouse.

!!! note "Environment-Specific"
    The storage account settings below are for demonstration. Consult your cloud/security team
    for your organization's required networking, encryption, and access configurations.

### Create a Storage Account

**1.** In the Azure Portal, search for **Storage accounts** and select it.

![Search Storage Accounts](../fivetran-iceberg/images/01-search-storage-accounts.png)

<br>

**2.** Click **Create**.

<img src="../fivetran-iceberg/images/02-click-create.png" alt="Click Create" width="55%">

<br>

**3.** Fill in the **Basics** tab:

| Setting | Value |
|---------|-------|
| **Subscription** | Your Azure subscription |
| **Resource group** | Create new or select existing |
| **Storage account name** | Must be globally unique (e.g., `timjonesiceberg`) |
| **Region** | East US 2 (match your Snowflake region if possible) |
| **Preferred storage type** | Azure Blob Storage or Azure Data Lake Storage Gen 2 |
| **Performance** | Standard |
| **Redundancy** | Locally-redundant storage (LRS) — adjust to your requirements |

Click **Next: Advanced**.

![Basics Tab](../fivetran-iceberg/images/03-basics-tab.png)

<br>

**4.** On the **Advanced** tab, leave all defaults **except**:

!!! danger "Critical Setting"
    **Enable hierarchical namespace** must be checked. This is what makes the storage account
    ADLS Gen2 rather than standard Blob Storage. Without this, the Iceberg integration will not work.

!!! tip
    Review all other settings on this tab with your internal security team.

![Advanced Tab](../fivetran-iceberg/images/04-advanced-tab.png)

<br>

**5.** On the **Networking** tab, all defaults were left as-is for this demo.

!!! note
    Review networking settings against your internal networking requirements. Your organization
    may require private endpoints, specific firewall rules, or restricted virtual network access.

![Networking Tab](../fivetran-iceberg/images/05-networking-tab.png)

<br>

**6.** On the **Data protection** tab, uncheck **Enable soft delete for file shares**.

This setting applies to Azure Files, which we are not using in this setup.

![Data Protection Tab](../fivetran-iceberg/images/06-data-protection-tab.png)

<br>

**7.** On the **Encryption** tab, all defaults were left as-is.

![Encryption Tab](../fivetran-iceberg/images/07-encryption-tab.png)

<br>

**8.** On the **Tags** tab, no tags were added for this demo.

!!! tip
    In production, add tags that align with your organization's requirements (e.g., `Environment`, `Owner`, `CostCenter`).

![Tags Tab](../fivetran-iceberg/images/08-tags-tab.png)

<br>

**9.** Navigate to **Review + create**, review all selections, and click **Create**.

<img src="../fivetran-iceberg/images/09-review-and-create.png" alt="Review and Create" width="75%">

<br>

**10.** Once deployment completes, click **Go to resource**.

<img src="../fivetran-iceberg/images/10-deployment-complete.png" alt="Deployment Complete" width="75%">

### Create a Container

<br>

**11.** In the left-hand pane, navigate to **Data storage** and select **Containers**.

![Navigate to Containers](../fivetran-iceberg/images/11-navigate-containers.png)

<br>

**12.** Click **+ Add container**.

![Add Container](../fivetran-iceberg/images/12-add-container.png)

<br>

**13.** Give the container a name (e.g., `landing`) and keep all defaults. Click **Create**.

![Name Container](../fivetran-iceberg/images/13-name-container.png)

!!! success "Storage Setup Complete"
    Your Azure Storage Account and ADLS Gen2 container are now provisioned and ready for Snowflake to use as external storage for Iceberg tables.

---

<!-- Remaining sections will be added as the tutorial is built out -->
