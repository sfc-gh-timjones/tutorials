# Snowflake Managed Iceberg Tables (Azure)

Configure Snowflake-managed Apache Iceberg tables using Azure ADLS Gen2 as external storage.

!!! quote "Source of Truth"
    This guide is based on the official Snowflake documentation. Always refer to the official docs as the source of truth, as steps may change over time:
    [Configure an external volume for Microsoft Azure](https://docs.snowflake.com/en/user-guide/tables-iceberg-configure-external-volume-azure)

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

## 2. Retrieve Your Azure Tenant ID

**14.** In the Azure Portal search bar, search for **Microsoft Entra ID** and select it from the dropdown.

![Search Microsoft Entra ID](images/14-search-microsoft-entra-id.png)

<br>

**15.** On the **Microsoft Entra ID** overview page, locate and copy the **Directory (tenant) ID**. Save this value — you will need it when creating the external volume in Snowflake.

![Copy Tenant ID](images/15-entra-id-tenant-id.png)


<br>

**15.** On the **Microsoft Entra ID** overview page, copy your **Directory (tenant) ID** and save it — you will need this when creating the External Volume in Snowflake.

![Entra ID Tenant ID](images/15-entra-id-tenant-id.png)

---

## 3. Create an External Volume in Snowflake

In Snowflake, open a **SQL worksheet**. We'll be creating an External Volume that points to your ADLS Gen2 container.

Reference documentation: [CREATE EXTERNAL VOLUME](https://docs.snowflake.com/en/sql-reference/sql/create-external-volume) — scroll to the **Microsoft Azure** section and use the **Data Lake Storage** URL format.

!!! abstract "What's happening"
    An External Volume tells Snowflake where your cloud storage lives and gives it the credentials needed to read and write Iceberg data files. Without this, Snowflake has no way to access your ADLS container.

**16.** Run the following SQL, replacing the placeholder values with your own:

!!! note "**You Can Also Use the Snowflake UI**"
    Rather than running SQL, you can configure the External Volume directly in the Snowflake web interface: [Configure an External Volume in the Snowflake UI](https://docs.snowflake.com/en/user-guide/tables-iceberg-configure-external-volume-azure?utm_source=chatgpt.com#configure-an-external-volume-in-sf-web-interface)

```sql
CREATE EXTERNAL VOLUME IF NOT EXISTS my_external_volume_name 
  STORAGE_LOCATIONS =
    (
      (
        NAME = 'your_chosen_name_here' -- This can be any value, it does not need to match the name of anything in Azure. It makes sense to name it the same as the external volume.
          STORAGE_PROVIDER = 'AZURE'
          AZURE_TENANT_ID = '<tenant_id>'
          STORAGE_BASE_URL = 'azure://<storage_account>.dfs.core.windows.net/<container>/' 
          --This should match the name of your storage account and container. Can include <path> after container name to provide granular control over logical directories in the container.
      )
    )
  ALLOW_WRITES = TRUE
  COMMENT = 'My external volume to connect to azure for Iceberg.';
```

**17.** Once created, run the following to describe your external volume:

```sql
DESCRIBE EXTERNAL VOLUME my_external_volume_name;
```

<br>

**18.** Immediately after, run the following query to extract the key properties from the result:

```sql
SELECT 
    PARSE_JSON("property_value"):AZURE_MULTI_TENANT_APP_NAME AZURE_MULTI_TENANT_APP_NAME,
    PARSE_JSON("property_value"):AZURE_CONSENT_URL AZURE_CONSENT_URL,
    PARSE_JSON("property_value"):NAME AS NAME,
    PARSE_JSON("property_value"):STORAGE_REGION STORAGE_REGION,
    PARSE_JSON("property_value"):AZURE_TENANT_ID AZURE_TENANT_ID
FROM TABLE(RESULT_SCAN(LAST_QUERY_ID()))
WHERE "property" = 'STORAGE_LOCATION_1';
```

!!! important "Copy These Values"
    From the query results, copy out the **`AZURE_CONSENT_URL`** and **`AZURE_MULTI_TENANT_APP_NAME`** — both will be needed for the remaining configuration steps.

    See: [Step 2: Grant Snowflake Access to the Storage Location](https://docs.snowflake.com/en/user-guide/tables-iceberg-configure-external-volume-azure#step-2-grant-snowflake-access-to-the-storage-location)

---

## 4. Grant Snowflake Access to the Storage Location

**19.** In the Azure Portal:

- Open your **storage account**
- Go to **Access control (IAM)**
- Click **Add role assignment**

![IAM Add Role Assignment](images/19-iam-add-role-assignment.png)

<br>

**20.** Search for **Storage Blob Data Contributor**, select it, and click **Next** at the bottom of the screen.

![Storage Blob Data Contributor](images/20-storage-blob-data-contributor.png)

<br>

**21.** Click **+ Select members**.

![Select Members](images/21-select-members.png)

<br>

**22.** Search for the Snowflake-generated service principal using the value from `AZURE_MULTI_TENANT_APP_NAME`, select it, then click **Review + assign** to finish.

!!! tip "Search Only the Prefix"
    Remove everything after the underscore in the `AZURE_MULTI_TENANT_APP_NAME` value — that portion is a timestamp. Only search for the text **before** the underscore.

![Search and Select Snowflake Principal](images/22-search-select-snowflake-principal.png)

<br>

**23.** Copy the `AZURE_CONSENT_URL` from the query results and paste it into your browser. Sign in as an Azure Admin and approve the application. This allows Snowflake's service principal to access your tenant.

Wait 1–2 minutes for Azure RBAC propagation before proceeding.

![Consent URL Approve](images/23-consent-url-approve.png)

<br>

**24.** Back in Snowflake, run the following to verify the external volume is working:

```sql
SELECT SYSTEM$VERIFY_EXTERNAL_VOLUME('my_external_volume_name') AS Test;
```

Ensure the output begins with `"success":true`.

!!! success "External Volume Ready"
    Your external volume is now fully configured. You're ready to start creating Snowflake-managed Iceberg tables that can be queried by Snowflake and other engines — including Apache Spark, Trino, and Flink — emphasizing the open, interoperable nature of the Iceberg format.
