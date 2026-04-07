# **Integrating Google Analytics with BigQuery**

## *Setup guide for Lemon AI*

This guide walks you through configuring Google Analytics and Google Cloud, so that Lemon AI can access your data and optimize your Google Ads campaigns. You can complete the setup yourself, or grant the Lemon AI team **Editor** access so we can assist you.

### **Prerequisites.**  Before you begin, make sure you have:

* A Google Analytics 4 (GA4) property with at least one active data stream.

* A Google Cloud account with billing enabled.

* Admin access to your [Google Analytics account](https://analytics.google.com) and [Google Cloud Console](https://console.cloud.google.com).

### **What you'll do.**  This guide covers three main tasks:

* Create a Google Cloud project and enable the BigQuery API.

* Link your Google Analytics property to BigQuery.

* Configure a service account, storage bucket, and access permissions in Google Cloud.


# **Step 1: Create a project in Google Cloud**

In this step, you create a Google Cloud project, which serves as the container for all BigQuery and Cloud Storage resources you set up later in this guide.

### **Create the project**

1. Go to the [Google Cloud Console](https://console.cloud.google.com).

2. In the top navigation bar, open the **project picker**, then click **New Project**

3. Enter a project name and click **Create**. The project appears in your project list.

![1](Images/LemonAI_image1.png)

![2](Images/LemonAI_image2.png)

### **Enable the BigQuery API**

1. In the left navigation menu, go to **APIs & Services** \> **Library**.

![3](Images/LemonAI_image3.png)

2. Search for **BigQuery API** and select it from the results.

3. Confirm that the status shows **API Enabled**. If not, click **Enable**.

![4](Images/LemonAI_image4.png)

# **Step 2: Link Google Analytics to BigQuery**

In this step, you connect your Google Analytics property to the Google Cloud project you created in Step 1\. This allows Analytics to export event data to BigQuery.

1. In your [Google Analytics account](https://analytics.google.com), click **Admin** (the gear icon in the bottom-left corner).

![5](Images/LemonAI_image5.png)

2. In the **Property** column, under **Product Links**, click **BigQuery Links**.

![6](Images/LemonAI_image6.png)

3. Click **Link**.

![7](Images/LemonAI_image7.png)

4. Click **Choose a BigQuery project**. A list of projects you have access to appears.

5. Search for the project you created in Step 1 and select it, then click **Confirm**.

6. Select a **data location**. You can choose any location that suits your business needs. If you don't have a preference, select europe-central2.

![8](Images/LemonAI_image8.png)

**Note:**  Remember the location you choose here — you'll need to use the same location when you create your Cloud Storage bucket in Step 3\.

7. Click **Next**.

8. Select **Configure data streams and events**, then select all data streams from the list.

**Event exclusions:**  Lemon AI uses only events that occurred after the user's first session or install, plus the install or click event itself. In the **Excluded events** section, select any events that don't reflect user behavior in your app or website.

**Advertising identifiers:**  If you have an Android app data stream and want to store advertising identifiers, select **Include advertising identifiers for mobile app streams**.

9. Click **Done**.

10. For **Export type**, select **Streaming**.

11. Click **Next**.

12. Review your settings and click **Submit**. The project now appears in the BigQuery Links section.

![9](Images/LemonAI_image9.png)

# **Step 3: Configure the integration in Google Cloud**

In this step, you create a service account, generate credentials, set up a Cloud Storage bucket, and grant the permissions that Lemon AI needs to access your data.

## **3a. Create a service account**

1. In the Google Cloud Console, go to **IAM & Admin** \> **Service Accounts**.

2. Click **Create service account**.

3. Fill in the following fields:

* **Name:** Enter your company or product name.

* **ID:** This is auto-populated from the name. You can't change it after creation.

* **Description:** Describe the purpose of the account. For example: Google Analytics exports to Lemon AI.

![10](Images/LemonAI_image10.png)

4. Click **Done**. The service account appears in the list.

## **3b. Create a service account key**

1. Click the service account you just created to open its details.

2. Go to the **Keys** tab.

3. Click **Add key** \> **Create new key**.

![11](Images/LemonAI_image11.png)

4. Select **JSON** as the key type and click **Create**. The key file downloads to your computer and also appears in the key list.

![12](Images/LemonAI_image12.png)

## **3c. Create a Cloud Storage bucket**

**Before you begin:**  Your Google Cloud account must have a billing method on file before you can create a Cloud Storage bucket. To add one, go to **Billing** \> **Add billing account** in the left navigation menu, or follow the link in the billing section of the Console.

![13](Images/LemonAI_image13.png)

1. In the left navigation menu, go to **Cloud Storage** \> **Buckets**.

![14](Images/LemonAI_image14.png)

2. Click **Create**.

3. Configure the bucket:

* **Location:** Select the same region you used when linking BigQuery in Step 2\. For example, if you chose europe-central2 there, select europe-central2 (Region) here. For more information, see [data export locations](https://cloud.google.com/bigquery/docs/exporting-data#data-locations).

* **Storage class, access control, and protection:** Leave these at the defaults — Standard, Uniform, and None.

4. Click **Create**.

![15](Images/LemonAI_image15.png)

## **3d. Create a Cloud Storage HMAC key**

1. In the left navigation menu, go to **Cloud Storage** \> **Settings**.

2. Click the **Interoperability** tab.

![16](Images/LemonAI_image16.png)

3. Click **Create a key for another service account**.

4. Select the service account you created in **section 3a**, then click **Create key**.

![17](Images/LemonAI_image17.png)

5. Copy and save both the **Access key** and the **Secret**. You'll share these with the Lemon AI team later. **The secret is shown only once.**

![18](Images/LemonAI_image18.png)

## **3e. Grant IAM permissions**

1. In the left navigation menu, go to **IAM & Admin** \> **IAM**.

2. Click **Grant Access**.

![19](Images/LemonAI_image19.png)

3. In the **New principals** field, enter the email address of the service account you created in section 3a.

![20](Images/LemonAI_image20.png)

4. Click **Add another role** and add the following three roles:

* **BigQuery Data Viewer**

* **BigQuery Job User**

* **Storage Object Admin** — grants full access to objects in Cloud Storage.

**Tip — restrict Storage Object Admin to your bucket only:**  To limit **Storage Object Admin** to the bucket you created (instead of all buckets), click **Add IAM condition** after selecting the role. In the condition editor, set the **Resource name** to projects/\_/buckets/\<BUCKET\_NAME\>, replacing \<BUCKET\_NAME\> with your bucket name — for example, projects/\_/buckets/lemonai\_123.

5. Click **Save**.

![21](Images/LemonAI_image21.png)

# **Step 4: Share credentials with Lemon AI**

After completing the steps above, share the following seven items with the Lemon AI team. Items 1–4 are available immediately. Item 5 (Dataset ID) becomes available approximately 24 hours after you complete the setup.

| \# | Credential / item | Description |
| ----- | :---- | :---- |
| **1** | **Service account key (JSON file)** | Downloaded in section 3b. Share the .json file directly. |
| **2** | **HMAC Access key & Secret** | Generated in section 3d. The secret is shown only once. |
| **3** | **Cloud Storage bucket name & location** | Configured in section 3c. |
| **4** | **BigQuery location** | The data location you selected in Step 2\. |
| **5** | **Dataset ID** | Available \~24 hours after setup. Instructions below. |
| **6** | **Measurement ID** | Found in Google Analytics Data Streams. Instructions below. |
| **7** | **Measurement Protocol API secret** | Created in Google Analytics Data Streams. Instructions below. |

Items 1–4 are files and values you already have from the previous steps. The following sections show you how to locate items 5, 6, and 7\.

## **Item 5: Find your Dataset ID**

The Dataset ID is created automatically by Google once the BigQuery link is active. It typically becomes available **approximately 24 hours** after you complete Step 2\.

1. In the Google Cloud Console, go to **BigQuery** \> **BigQuery Studio**.

![22](Images/LemonAI_image22.png)


2. In the left panel, locate your dataset — it appears at the top of the resource tree. The name follows the format analytics\_XXXXXXXXX, for example analytics\_1234567.  

![23](Images/LemonAI_image23.png)

3. Share this Dataset ID with the Lemon AI team.

## **Item 6: Find your Measurement ID**

1. In your [Google Analytics account](https://analytics.google.com), click **Admin** (the gear icon in the bottom-left corner).

![24](Images/LemonAI_image24.png)

2. In the **Property** setting, under **Data Collection and Modification**, click **Data Streams**.

3. Select your data stream from the list.

4. The Measurement ID appears in the top-right corner of the stream details panel. It follows the format G-XXXXXXXXXX, for example G-ABCD1234.

![25](Images/LemonAI_image25.png)

5. Share this Measurement ID with the Lemon AI team.

**For app data streams:**  Instead of a Measurement ID, share the **Stream ID** and the **Firebase App ID** with the Lemon AI team.

## **Item 7: Create a Measurement Protocol API secret**

Complete this in the same Data Streams screen you used in Item 6\.

1. In the Web stream details panel, click **Measurement Protocol API secrets**.

![26](Images/LemonAI_image26.png)

2. Click **Create**.

3. Enter a nickname for the secret. You can use any name — for example, yourcompany\_MP.

4. Click **Create**. The secret value appears in the centre of the screen, for example, 1a2b345cd.

![27](Images/LemonAI_image27.png)

5. Share this API secret with the Lemon AI team.

**Firebase app setup:**  If you're integrating an Android app, also go to **Firebase** \> **Project Settings** \> **Integrations** \> **BigQuery** and confirm that **Include Advertising Identifiers in Export** is enabled.

**Setup is complete.**

Your Google Analytics and BigQuery integration is complete. Lemon AI can now access your data and begin optimizing your Google Ads campaigns.
