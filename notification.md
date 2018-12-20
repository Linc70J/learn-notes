# Laravel Notification
Notification支持在各種傳遞渠道發送通知。
  - E-mail
  - SMS(通過Nexmo)
  - DataBase
  - Slack

### 快速建立Notification
在Laravel中，每個通知都是一個Class，可以使用Laravel Artisan快速生成。
```php
php artisan make:notification InvoicePaid
```

### 使用Models傳遞通知
需要在「**Models**」裡面 use **Illuminate\Notifications\Notifiable** Trait(並不僅限於User model)
```php
use Notifiable;
```

指定傳送通知給某個User：
```php
$user->notify(new InvoicePaid($invoice));
```

指定傳送通知給多個人：
```php
Notification::send($users, new InvoicePaid($invoice));
```

### 更直接的傳遞通知

直接指定傳送通知對象：
```php
Notification::route('mail', 'taylor@example.com')
            ->route('nexmo', '5555555555')
            ->notify(new InvoicePaid($invoice));
```

### 設定通知的方法
每個Notification class都有一個via方法，用於確定通知將在哪些渠道上傳遞。([更多的通知管道](http://laravel-notification-channels.com/))。
```php
public function via($notifiable)
{
    return return [GcmChannel::class];
}
```

###自定義Models通知對象
以下為**Illuminate\Notifications\Notifiable** Trait 預設方法：
```php
public function routeNotificationFor($driver)
{
    if (method_exists($this, $method = 'routeNotificationFor'.Str::studly($driver))) {
        return $this->{$method}();
    }

    switch ($driver) {
        case 'database':
            return $this->notifications();
        case 'mail':
            return $this->email;
        case 'nexmo':
            return $this->phone_number;
    }
}
```

自定義E-mail來源：
```php
public function routeNotificationForMail($notification)
{
    return $this->email_address;
}
```

### 通知內容設定
在 Notification class 定義對各個渠道上傳遞內容的 function，例如：toMail、toDatabase、toNexmo、toBroadcast。

### E-mail的通知內容：
自定義通知內容：
```php
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        'emails.name', ['invoice' => $this->invoice]
    );
}
```

格式化內容：
```php
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
```

自定義格式化模板：
```php
php artisan vendor:publish --tag=laravel-notifications
```

### Database的通知內容：
需要先建立 Database 訊息儲存的 Table：
```php
php artisan notifications:table
php artisan migrate
```

將資訊儲存至對應欄位：
```php
public function toDatabase($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

取得所有通知：
```php
foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

取得未讀通知：
```php
foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

標記為已讀：
```php
foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

### 使用Queue來通知
```php
class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

### 設定通知的Event
發送通知時，通知系統會觸發該事件。可在以下位置註冊此活動的Listener：Illuminate\Notifications\Events\NotificationSentEventServiceProvider
```php
protected $listen = [
    'Illuminate\Notifications\Events\NotificationSent' => [
        'App\Listeners\LogNotification',
    ],
];
```

也可以反過來使用，讓Event觸發時發送通知。

#### 多國語系
可以使用以下方法設定通知內容所使用的語系：
```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```
或
```php
Notification::locale('es')->send($users, new InvoicePaid($invoice));
```
