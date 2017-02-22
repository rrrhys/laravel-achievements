# Laravel Achievements

***FORKED from https://github.com/gstt/laravel-achievements***
An implementation of an Achievement System in Laravel, inspired by Laravel's Notification system. 

## Table of Contents
1. [Requirements](#requirements)
2. [Installation](#installation)
3. [Creating Achievements](#creating)
4. [Unlocking Achievements](#unlocking)
5. [Adding Progress](#progress)
6. [Retrieving Achievements](#retrieving)
7. [License](#license)

## Requirements <a name="requirements"></a>

- Laravel 5.3 or higher

## Installation <a name="installation"></a>

Default installation is via [Composer](https://getcomposer.org/).

```
composer require gstt/laravel-achievements
```

Add the Service Provider to your `config/app` file in the `providers` section.

```php
'providers' => [
    ...
    Gstt\Achievements\AchievementsServiceProvider::class,
```

Backup your database and run the migrations in order to setup the required tables on the database.

```
php artisan migrate
```

## Creating Achievements <a name="creating"></a>
Similar to Laravel's implementation of [Notifications](https://laravel.com/docs/5.4/notifications), each Achievement is 
represented by a single class (typically stored in the `app\Achievements` directory.) This directory will be created 
automatically for you when you run the `make:achievement` command.

```
php artisan make:achievement UserMadeAPost
```
This command will put a fresh Achievement in your `app/Achievements` directory with only has two properties defined: 
`name` and `description`. You should change the default values for these properties to something that better explains
what the Achievement is and how to unlock it. When you're done, it should look like this:

```php
<?php

namespace App\Achievements;

use Gstt\Achievements\Achievement;

class UserMadeAPost extends Achievement
{
    /*
     * The achievement name
     */
    public $name = "Post Created";

    /*
     * A small description for the achievement
     */
    public $description = "Congratulations! You have made your first post!";
}
```

## Unlocking Achievements <a name="unlocking"></a>
Achievements can be unlocked by using the `Achiever` trait. Let's explore the usage:

```php
<?php

namespace App;

use Gstt\Achievements\Achiever;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Achiever;
}
```
This trait contains the `unlock` method, that can be used to unlock an Achievement. The `unlock` method expects an 
`Achievement` instance:

```php
use App\Achievements\UserMadeAPost

$user->unlock(new UserMadeAPost());
```
Remember that you're not restricted to the `User` class. You may add the `Achiever` trait to any entity that could
unlock Achievements.

## Adding Progress

Instead of directly unlocking an achievement, you can add a progress to it. For example, you may have an achievement 
`UserMade10Posts` and you want to keep track of how the user is progressing on this Achievement.

In order to do that, you must set an additional parameter on your `UserMade10Posts` class, called `$points`:

```php
<?php

namespace App\Achievements;

use Gstt\Achievements\Achievement;

class UserMade10Posts extends Achievement
{
    /*
     * The achievement name
     */
    public $name = "10 Posts Created";

    /*
     * A small description for the achievement
     */
    public $description = "Wow! You have already created 10 posts!";
    
    /*
     * The amount of "points" this user need to obtain in order to complete this achievement
     */
    public $points = 10;
}
```
You may now control the progress by the methods `addProgress` and `removeProgress` on the `Achiever` trait. 
Both methods expect an `Achievement` instance and an amount of points to add or remove:

```php
use App\Achievements\UserMade10Posts;

$user->addProgress(new UserMade10Posts(), 1); // Adds 1 point of progress to the UserMade10Posts achievement
```

In addition, you can use the methods `resetProgress` to set the progress back to 0 and `setProgress` to set it to a 
specified amount of points:

```php
use App\Achievements\FiveConsecutiveSRanks;

if($rank != 'S'){
    $user->resetProgress(new FiveConsecutiveSRanks());
} else {
    $user->addProgress(new FiveConsecutiveSRanks(), 1);
}
```

```php
use App\Achievements\Have1000GoldOnTheBag;

$user->setProgress(new Have100GoldOnTheBag(), $user->amountOfGoldOnTheBag);
```

Once an Achievement reach the defined amount of points, it will be automatically unlocked.

## Retrieving Achievements <a name="retrieving"></a>
The `Achiever` trait also adds a convenient relationship to the entity implementing it: `achievements()`. You can use it
to retrieve progress for all achievements the entity has interacted with. Since `achievements()` is a relationship, you
can use it as a QueryBuilder to filter data even further.

```php
$achievements   = $user->achievements;
$unlocked_today = $user->achievements()->where('unlocked_at', '>=', Carbon::yesterday())->get();
```

You can also search for a specific achievement using the `achievementStatus()` method.

```php
$details = $user->achievementStatus(new UserMade10Posts());
```

There are also two additional helpers on the `Achiever` trait: `inProgressAchievements()` and `unlockedAchievements()`.

## License <a name="license"></a>

Laravel Achievements is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).
