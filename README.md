# BillingHelper

#### 集成

- 在app build.gradle添加依赖

```
implementation 'com.github.Geekince:BillingPay:1.0.2'
```

#### 使用

- SDK包含了2种支付方式。一种是直接调用谷歌原生应用内支付，一种是传递url进行H5支付

1. 谷歌应用内支付

```

prvate BillingManager billingManager;

// 1. 创建回调
BaseBillingUpdateListener billingUpdateListener = new SimpleBillingUpdateListener() {
    @Override
    public void onBillingClientSetupFinished() {
        // 3. 在与Google Play建立好连接后，进行支付
        if (billingManager != null) {
            billingManager.launchBillingFlow("7days", BillingClient.SkuType.SUBS);
        }
    }

    @Override
    public void onPurchasesUpdated(List<Purchase> purchases) {
        // 此处为支付成功回调，可以拿到支付成功的商品调用我们自己的业务接口来对商品购买状态进行二次校验
        ...
        // 如果后台通过二次校验并为该用户开通了权限或发放了对应的虚拟商品，则客户端需要做的是对该笔订单进行确认
        // 注：若客户端不调用该方法，则必须由后端进行acknowledge()方法的调用来确认订单，否则不确认订单订单将会在3天后自动进行退款
        
        // 消耗型商品订单确认api：billingManager.consumeAsync(String purchaseToken, String payload);
        // 非消耗型商品订单确认api：billingManager.acknowledgePurchase(String purchaseToken, String payload);
    }

    @Override
    public void onPurchasesCancel() {
        Toast.makeText(MainActivity.this, "取消购买", Toast.LENGTH_SHORT).show();
        Log.e(TAG, "取消购买");
    }

    @Override
    public void onPurchasesFailure(int errorCode, String message) {
        Toast.makeText(MainActivity.this,
                "购买失败[code：" + errorCode + ",message：" + message + "]", Toast.LENGTH_SHORT).show();
        Log.e(TAG, "购买失败[code：" + errorCode + ",message：" + message + "]");
    }
};

// 2. 进行与Google Play连接
public void googleBilling(){
    billingManager = new BillingManager(this, billingUpdateListener);
    billingManager.startServiceConnection(null);
}

// 4. 释放资源
@Override
protected void onDestroy() {
    super.onDestroy();
    if (billingManager != null){
        billingManager.destroy();
    }
}
```

2. H5支付（包含国外第三方支付）

```
prvate BillingManager billingManager;

BillingPurchasesReceiver billingPurchasesReceiver = new BillingPurchasesReceiver() {

    @Override
    public void onPurchasesUpdated(PurchaseInfo purchaseInfo) {
        // 此处为支付成功回调，可以拿到支付成功的商品调用我们自己的业务接口来对商品购买状态进行二次校验
        ...
        // 如果后台通过二次校验并为该用户开通了权限或发放了对应的虚拟商品，则客户端需要做的是对该笔订单进行确认
        // 注：若客户端不调用该方法，则必须由后端进行acknowledge()方法的调用来确认订单，否则不确认订单订单将会在3天后自动进行退款
        
        // 消耗型商品订单确认api：billingManager.consumeAsync(String purchaseToken, String payload);
        // 非消耗型商品订单确认api：billingManager.acknowledgePurchase(String purchaseToken, String payload);
        
        Toast.makeText(MainActivity.this, "购买成功", Toast.LENGTH_SHORT).show();
        Log.e(TAG, "购买成功：" + purchaseInfo.toString());
    }

    @Override
    public void onPurchasesCancel() {

        Toast.makeText(MainActivity.this, "取消购买", Toast.LENGTH_SHORT).show();
        Log.e(TAG, "取消购买");
    }

    @Override
    public void onPurchasesFailure(int errorCode, String message) {
        Toast.makeText(MainActivity.this,
                "购买失败[code：" + errorCode + ",message：" + message + "]", Toast.LENGTH_SHORT).show();
        Log.e(TAG, "购买失败[code：" + errorCode + ",message：" + message + "]");
    }
};

// 2. 发起H5支付
public void purchase() {
    BillingManager billingManager = new BillingManager(this, billingPurchasesReceiver);
    billingManager.launchBillingFlow("<三方H5支付网址>");
}

// 3. 释放资源
@Override
protected void onDestroy() {
    super.onDestroy();
    if (billingManager != null){
        billingManager.destroy();
    }
}
```

> 说明：
>     
>     对于原生支付，无论是进行商品查询操作还是购买操作，必须是在与Google Play建立连接后进行操作。即在收到onBillingClientSetupFinished()回调后进行操作。
>     
>     Google Billing应用内支付2.0及以上版本，必须在3天内确认购买交易，如果未进行正确确认，将导致系统对相应购买交易按退款处理。
>     
>     对于H5支付，URL可以是我们业务的链接地址(比如我们的VIP支付项页面)，也可以直接是国外第三方支付（参数完整的）链接地址。
>     
>     谷歌支付需要提前在谷歌开发者平台配置测试账号以及其他配置，才能进行SDK调用。由于该库是从公司项目抽离。demo没有使用项目包名，运行无法得到预期效果。

- 混淆

SDK已经增加了混淆配置，原则上无须添加以下配置。若混淆配置合并失败，请手动添加

```
-keep class com.pay.android.billing.model.** { *; }
-keep class com.android.vending.billing.** { *; }

参考：

- https://github.com/pengfeigao/GoogleBilling
