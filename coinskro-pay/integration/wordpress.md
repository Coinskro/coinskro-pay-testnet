---
layout: default
title: WordPress
parent: Integrations
nav_order: 1
permalink: /coinskro-pay/integration/wordpress
---

# Coinskro Payment Gateway for WooCommerce - Documentation

Welcome to the official documentation for the **Coinskro Payment Gateway** plugin for WooCommerce. This guide will walk you through the entire process from installing the plugin to successfully processing a payment via the Pi Network.

---

## 1. Prerequisites

Before installing the Coinskro Payment Gateway, please ensure your WordPress site meets the following requirements:

- **WordPress**: Version 5.8 or higher.
- **WooCommerce**: Version 6.0 or higher.
- **PHP**: Version 7.2 or higher.
- An active **Coinskro Merchant Account** to obtain your API keys.

---

## 2. Installation

You can download the official plugin from the WordPress repository:
[Download Coinskro Payment Gateway plugin](#) _(Link coming soon)_

### Method 1: Uploading the Plugin ZIP

1. Download the `coinskro.zip` plugin file.
2. Log in to your WordPress Admin Dashboard.
3. Navigate to **Plugins > Add New** and click the **Upload Plugin** button at the top of the page.
4. Click **Choose File**, select the `coinskro.zip` file, and click **Install Now**.
5. Once the installation is complete, click **Activate Plugin**.

### Method 2: Manual FTP Upload

1. Extract the `coinskro.zip` file on your local computer.
2. Connect to your web server using an FTP client (like FileZilla).
3. Navigate to your WordPress installation's plugin directory (`/wp-content/plugins/`).
4. Upload the extracted `coinskro` folder into the `plugins` directory.
5. Log in to your WordPress Admin Dashboard, navigate to **Plugins**, locate **Coinskro - Secure your Assets in transactions**, and click **Activate**.

---

## 3. Configuration & Setup

Upon activating the plugin, you will see a welcome notification in your WooCommerce Inbox. To start accepting payments, you must configure your Coinskro API keys.

1. In your WordPress Admin Dashboard, go to **WooCommerce > Settings**.
2. Click on the **Payments** tab.
3. Locate **Coinskro** in the list of payment gateways and toggle the switch to enable it.
4. Click the **Manage** button (or **Settings** from the Plugins page) to open the Coinskro configuration page.

### Configuring the Settings

Fill out the form with your preferences and Coinskro API details:

- **Enable/Disable**: Ensure the "Enable Coinskro" checkbox is checked.
- **Environment Settings**: Choose between `Testnet` (for sandbox testing) and `Mainnet` (for live production transactions).
- **Secret Key**: Enter the Secret Key provided in your Coinskro Merchant Dashboard. Be sure to use your Testnet key for Testnet and Mainnet key for Mainnet.
- **Payment Flow Mode**: Select whether the customer should be redirected to the Coinskro portal or if the payment should open in a Popup modal (if applicable).
- _(Optional)_ **Success/Failure URLs**: Define custom pages where the customer should be redirected after a transaction succeeds or fails.

5. Click **Save changes** at the bottom of the screen.

---

## 4. The Customer Checkout Experience

The Coinskro plugin fully supports both the traditional WooCommerce Shortcode Checkout and the modern WooCommerce Block Checkout.

### Step 1: Adding a Product to the Cart

Customers browse your store and add one or more products to their cart as usual.

### Step 2: Proceeding to Checkout

When the customer navigates to the Checkout page, they will fill out their billing and shipping details.

### Step 3: Selecting Coinskro Pay

In the Payment Methods section, the customer will see **Coinskro Pay** listed (alongside the official Coinskro logo).

### Step 4: Making the Payment

1. The customer clicks **Place Order**.
2. Based on your configuration, they are redirected securely to the Coinskro Payment Portal.
3. The customer authenticates and authorizes the transaction using the Pi Network infrastructure.

### Step 5: Verification & Completion

1. Once the payment is authorized on Coinskro, the customer is immediately redirected back to your WooCommerce store.
2. The Coinskro plugin automatically verifies the transaction status in the background using the secure API and updates the WooCommerce Order Status.
3. The customer sees the standard "Order Received" success page.
4. The store administrator can view the transaction details inside the WooCommerce order notes.

---

## Troubleshooting

- **Missing Settings Link?** Ensure that your plugin is activated correctly. You can always reach the settings manually via **WooCommerce > Settings > Payments > Coinskro**.
- **Transactions failing verification?** Double-check that your Secret Key matches your selected Environment (Testnet vs. Mainnet) and that the Coinskro webhook URL (if applicable) is correctly pointing to your store.
