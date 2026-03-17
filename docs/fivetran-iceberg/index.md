# Fivetran + Iceberg + Snowflake (Azure)

Connect Fivetran to Azure ADLS Gen2 as an Apache Iceberg destination, then integrate with Snowflake to query open lakehouse data using a catalog-linked database.

!!! warning "Adapt to Your Environment"
    This guide is a reference implementation demonstrated in a simplified environment.
    Your networking, security policies, IAM configurations, and infrastructure will differ.
    Always validate against your organization's requirements before implementing.

## Architecture

```
Azure SQL Database
       │
       ▼
   Fivetran
       │
       ▼
  Azure ADLS Gen2  ◄──  Iceberg Tables
       │
       ▼
   Snowflake
   (Catalog-Linked Database)
```

## Prerequisites

- Azure account with permissions to create storage accounts, app registrations, and role assignments
- Fivetran account with permissions to configure connectors and destinations
- Snowflake account (Enterprise edition or higher) with `ACCOUNTADMIN` role
- An Azure SQL Database with data to sync (or another supported Fivetran source)

---

## 1. Provision Cloud Storage (Azure ADLS)

Create an Azure Storage Account with ADLS Gen2 to store Iceberg tables written by Fivetran.

!!! note "Environment-Specific"
    The storage account settings below are for demonstration. Consult your cloud/security team
    for your organization's required networking, encryption, and access configurations.

### Create a Storage Account

**1.** In the Azure Portal, search for **Storage accounts** and select it.

![Search Storage Accounts](images/01-search-storage-accounts.png)

**2.** Click **Create**.

![Click Create](images/02-click-create.png)

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

![Basics Tab](images/03-basics-tab.png)

**4.** On the **Advanced** tab, leave all defaults **except**:

!!! danger "Critical Setting"
    **Enable hierarchical namespace** must be checked. This is what makes the storage account
    ADLS Gen2 rather than standard Blob Storage. Without this, the Iceberg integration will not work.

!!! tip
    Review all other settings on this tab with your internal security team.

![Advanced Tab](images/04-advanced-tab.png)

**5.** On the **Networking** tab, all defaults were left as-is for this demo.

!!! note
    Review networking settings against your internal networking requirements. Your organization
    may require private endpoints, specific firewall rules, or restricted virtual network access.

![Networking Tab](images/05-networking-tab.png)

**6.** On the **Data protection** tab, uncheck **Enable soft delete for file shares**.

This setting applies to Azure Files, which we are not using in this setup.

![Data Protection Tab](images/06-data-protection-tab.png)

**7.** On the **Encryption** tab, all defaults were left as-is.

![Encryption Tab](images/07-encryption-tab.png)

**8.** On the **Tags** tab, no tags were added for this demo.

!!! tip
    In production, add tags that align with your organization's requirements (e.g., `Environment`, `Owner`, `CostCenter`).

![Tags Tab](images/08-tags-tab.png)

**9.** Navigate to **Review + create**, review all selections, and click **Create**.

![Review and Create](images/09-review-and-create.png)

**10.** Once deployment completes, click **Go to resource**.

![Deployment Complete](images/10-deployment-complete.png)

### Create a Container

**11.** In the left-hand pane, navigate to **Data storage** and select **Containers**.

![Navigate to Containers](images/11-navigate-containers.png)

**12.** Click **+ Add container**.

![Add Container](images/12-add-container.png)

**13.** Give the container a name (e.g., `landing`) and keep all defaults. Click **Create**.

![Name Container](images/13-name-container.png)

!!! success "Storage Setup Complete"
    Your Azure Storage Account and ADLS Gen2 container are now provisioned and ready for Fivetran to write Iceberg data.

---

## 2. Enable Fivetran Access to ADLS

Create an Azure service principal so Fivetran can write Iceberg data to your ADLS container.

<!-- Screenshots and steps will be added as the user walks through the setup -->

---

## 3. Configure Fivetran Data Pipeline

Set up the Fivetran connector and destination to sync data from Azure SQL Database to ADLS as Iceberg tables.

<!-- Screenshots and steps will be added as the user walks through the setup -->

---

## 4. Connect Snowflake to ADLS Iceberg Data

Create a Snowflake external volume pointing to the ADLS container and grant the necessary Azure permissions.

<!-- Screenshots and steps will be added as the user walks through the setup -->

---

## 5. Integrate the Fivetran Iceberg Catalog

Create a Snowflake catalog integration pointing to the Fivetran Iceberg REST catalog.

<!-- Screenshots and steps will be added as the user walks through the setup -->

---

## 6. Create a Catalog-Linked Database

Create a catalog-linked database in Snowflake that auto-discovers schemas and tables from the Fivetran catalog.

<!-- Screenshots and steps will be added as the user walks through the setup -->

---

## 7. Query Iceberg Tables from Snowflake

Use standard SQL to query Iceberg tables stored in ADLS — no data copying required.

<!-- Screenshots and steps will be added as the user walks through the setup -->
