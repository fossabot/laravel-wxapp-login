# Laravel 快速接入微信小程序
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fxiaohuilam%2Flaravel-wxapp-login.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fxiaohuilam%2Flaravel-wxapp-login?ref=badge_shield)

---

## 安装
```bash
composer require xiaohuilam/laravel-wxapp-login -vvv
```

## 发布
执行以下命令发布路由和 controller
```bash
php artisan vendor:publish --tag=wechat-login
```

## 表结构、模型改动
laravel-wxapp-login 需要在 `users` 表中添加 `openid` 的字段，所以需要运行 [`database/migrations/2019_05_28_060312_users_add_openid.php`](https://github.com/xiaohuilam/laravel-wxapp-login/blob/master/publishes/migrations/2019_05_28_060312_users_add_openid.php)
```bash
php artisan migrate
```

另外，需要在用户模型中，将 `openid` 加入到 `$fillable` 属性:

打开 `app\User.php` （如果你修改过模型位置，请以自己项目实际位置为准）
```php
class User extends Authenticatable
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        //... 你原先的字段
        'openid',
    ];
}
```

## 配置
```env
WECHAT_MINI_PROGRAM_APPID=#小程序的appid
WECHAT_MINI_PROGRAM_SECRET=#小程序的secret
```

## 使用

### 接口使用

在 `routes/wechat.php`
```php
Route::group(['middleware' => ['guest']], function () {
    Route::post('login', 'Auth\\WechatLoginController@login')->name('wechat.login');
    //这里放未登录的api
});

Route::group(['middleware' => ['auth:wechat']], function () {
    //这里放置你的需要登录的 api 路由，如用户资料API、修改资料API...
});
```

### 修改注册用户逻辑

在 `app/Http/Controllers/Auth/WechatLoginController.php` 中修改

方法为：
- `protected function registerUser($credential)` 完整的注册逻辑
- `protected function userAttributes($credential)` 用仅需修改用户属性时，只需覆盖此方法即可


### 在小程序JS使用

**Restful API**

在
```bash
php artisan tinker
```
运行 `route('wechat.login')` 得到微信登录的 api 的 URL， 调用微信登录
```javascript
let url = '上面获得的url'
wx.login({
  success: (res) => {
      wx.request({
        url: url,
        method: 'POST',
        data: {
            code: res.code
        },
        headers: {
            Authorization: null
        },
        success: (response) => {
            // 这里拿到的token = response.data.token，给后面所有需要登录的api都带上 {headers: {Authorization: response.data.token}}
        },
      })
  },
})
```

**Websocket**

小程序代码
```javascript
import Echo from 'laravel-echo'
let io = require('weapp.socket.io')

const echo = new Echo({
  client: io,
  broadcaster: 'socket.io',
  host: '??',
  auth: {
    headers: {
      Authorization: 'Bearer ' + '上面取得的 token',
    },
  },
});

echo.private(`channel.${user_id}`)
  .listen(`broadcastAs 后的名字`, (e) => {
    console.log(e);
  });
```

注册 channel.php
```php
Broadcast::channel('channel.{id}', function ($user, $id) {
    return $user->id == $id;
}, ['guards' => ['wechat',]]); //一定要指定 guards!

```

## 授权
MIT

## 鸣谢
- Overtrue's [laravel-easywechat](https://github.com/overtrue/laravel-wechat)


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fxiaohuilam%2Flaravel-wxapp-login.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fxiaohuilam%2Flaravel-wxapp-login?ref=badge_large)