# Laravel Like

👍 User-like features for Laravel Application.

## Installing

```shell
composer require eslamfaroug/laravel-like-dislike
```

### Configuration

This step is optional

```php
php artisan vendor:publish
```

### Migrations

This step is also optional, if you want to custom likes table, you can publish the migration files:

```php
php artisan vendor:publish
```

## Usage

### Traits

#### `EslamFaroug\LaravelLikeDislike\Traits\Liker`

```php

use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use EslamFaroug\LaravelLikeDislike\Traits\Liker;

class User extends Authenticatable
{
    use Liker;

    <...>
}
```

#### `EslamFaroug\LaravelLikeDislike\Traits\Likeable`

```php
use Illuminate\Database\Eloquent\Model;
use EslamFaroug\LaravelLikeDislike\Traits\Likeable;

class Post extends Model
{
    use Likeable;

    <...>
}
```

### API

```php
$user = User::find(1);
$post = Post::find(2);

$user->like($post);
$user->unlike($post);
$user->toggleLike($post);

$user->hasLiked($post);
$post->isLikedBy($user);

$user->dislike($post);
$user->undislike($post);
$user->toggleDislike($post);

$user->hasDisliked($post);
$post->isDislikedBy($user);
```

Get user likes with pagination:

```php
$likes = $user->likes()->with('likeable')->paginate(20);

foreach ($likes as $like) {
    $like->likeable; // App\Post instance
}

$dislikes = $user->dislikes()->with('likeable')->paginate(20);

foreach ($dislikes as $dislike) {
    $dislike->likeable; // App\Post instance
}
```

Get object likers:

```php
foreach($post->likers as $user) {
    // echo $user->name;
}

foreach($post->dislikers as $user) {
    // echo $user->name;
}
```

with pagination:

```php
$likers = $post->likers()->paginate(20);

foreach($likers as $user) {
    // echo $user->name;
}

$dislikers = $post->dislikers()->paginate(20);

foreach($dislikers as $user) {
    // echo $user->name;
}
```

### Aggregations

```php
// Likes
// all
$user->likes()->count();

// with type
$user->likes()->withType(Post::class)->count();

// likers count
$post->likers()->count();

// Dislikes
// all
$user->dislikes()->count();

// with type
$user->dislikes()->withType(Post::class)->count();

// likers count
$post->dislikers()->count();
```

List with `*_count` attribute:

```php
// likes_count
$users = User::withCount('likes')->get();

foreach($users as $user) {
    // $user->likes_count;
}

// dislikes_count
$users = User::withCount('dislikes')->get();

foreach($users as $user) {
    // $user->dislikes_count;
}

// likers_count
$posts = User::withCount('likers')->get();

foreach($posts as $post) {
    // $post->likes_count;
}

// dislikers_count
$posts = User::withCount('dislikers')->get();

foreach($posts as $post) {
    // $post->dislikes_count;
}
```

### N+1 issue

To avoid the N+1 issue, you can use eager loading to reduce this operation to just 2 queries. When querying, you may specify which relationships should be eager loaded using the `with` method:

```php
// Liker
$users = App\User::with('likes')->get();

foreach($users as $user) {
    $user->hasLiked($post);
}

// Disliker
$users = App\User::with('dislikes')->get();

foreach($users as $user) {
    $user->hasDisliked($post);
}

// Likeable
$posts = App\Post::with('likes')->get();
// or
$posts = App\Post::with('likers')->get();

foreach($posts as $post) {
    $post->isLikedBy($user);
}

$posts = App\Post::with('dislikes')->get();
// or
$posts = App\Post::with('dislikers')->get();

foreach($posts as $post) {
    $post->isDislikedBy($user);
}
```

Of course we have a better solution, which can be found in the following section：

### Attach user like status to likeable collection

You can use `Liker::attachLikeStatus($likeables)` to attach the user like status, it will attach `has_liked` attribute to each model of `$likeables`:

#### For model
```php
$post = Post::find(1);

$post = $user->attachLikeStatus($post);

// result
[
    "id" => 1
    "title" => "Add socialite login support."
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_liked" => true
 ],

$post = Post::find(1);

$post = $user->attachDislikeStatus($post);

// result
[
    "id" => 1
    "title" => "Add socialite login support."
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_disliked" => true
 ],
```

#### For `Collection | Paginator | LengthAwarePaginator | array`:

```php
$posts = Post::oldest('id')->get();

$posts = $user->attachLikeStatus($posts);

$posts = $posts->toArray();

// result
[
  [
    "id" => 1
    "title" => "Post title1"
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_liked" => true
  ],
  [
    "id" => 2
    "title" => "Post title2"
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_liked" => fasle
  ],
  [
    "id" => 3
    "title" => "Post title3"
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_liked" => true
  ],
]
```

```php
$posts = Post::oldest('id')->get();

$posts = $user->attachDislikeStatus($posts);

$posts = $posts->toArray();

// result
[
  [
    "id" => 1
    "title" => "Post title1"
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_disliked" => true
  ],
  [
    "id" => 2
    "title" => "Post title2"
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_disliked" => fasle
  ],
  [
    "id" => 3
    "title" => "Post title3"
    "created_at" => "2021-05-20T03:26:16.000000Z"
    "updated_at" => "2021-05-20T03:26:16.000000Z"
    "has_disliked" => true
  ],
]
```

#### For pagination

```php
$posts = Post::paginate(20);

$user->attachLikeStatus($posts);

$posts = Post::paginate(20);

$user->attachDislikeStatus($posts);
```

## Troubleshooting

If you encounter any issues or need help, please refer to the [Troubleshooting section](#troubleshooting) in the documentation for assistance.

## Contributing

Contributions are welcome! If you'd like to contribute to

the Laravel Like System Package, please follow the guidelines in the [Contributing section](#contributing) of the documentation.

## License

The Laravel Like System Package is open-source software licensed under the [MIT license](LICENSE).

---

This concludes the documentation for the Laravel Like System Package. For more information and detailed usage instructions, please refer to the sections above. If you have any questions or need further assistance, don't hesitate to reach out to the package author or community for support.

We hope you find the Laravel Like System Package a valuable addition to your Laravel projects! Happy coding!
