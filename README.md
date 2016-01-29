# Laravel Moderation
A simple Moderation System for Laravel 5.* that allows you to Approve or Reject resources like posts, comments, users, etc.

Keep your application pure by preventing offensive, irrelevant, or insulting content.

##Possible Use Case

1. User creates a resource (a post, a comment or any Eloquent Model).
2. The resource is pending and invisible in website (ex. `Post::all()` returns only approved posts).
3. Admin/Moderator/etc decides if the resource will be approved or rejected.

  1. **Approved**: Resource is now public and queryable.
  2. **Rejected**: Resource will be excluded from all queries. Rejected resources will be returned only if you scope a query to include them. (scope: `withRejected`)

4. You application is clean.

##Installation

First, install the package through Composer.

```php
composer require hootlex/laravel-moderation
```

Then include the service provider inside `config/app.php`.

```php
'providers' => [
    ...
    Hootlex\Friendships\ModerationServiceProvider::class,
    ...
];
```
Lastly you publish the config file.

```
php artisan vendor:publish --provider="Hootlex\Friendships\ModerationServiceProvider" --tag=config
```


## Setup the Model(s)

To enable moderation for a model, use the `Hootlex\Moderation\Moderatable` trait on the model and add the `status` and `moderated_at` columns to your model's table.
```php
use Hootlex\Moderation\Moderatable;
class Post extends Model
{
    use Moderatable;
    ...
}
```

Create a migration to add the 2 new columns. [(You can use custom names for the moderation columns)](#configuration)

Example Migration:
```php
class AddModeratioColumnsToPostsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('posts', function (Blueprint $table) {
            $table->smallInteger('status')->default(0);
            $table->dateTime('moderated_at')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('posts', function(Blueprint $table)
        {
            $table->dropColumn('status');
            $table->dropColumn('moderated_at');
        });
    }
}
```

**You are ready to go!**

##Usage
> **Note:** In next examples I will use Post model to demonstrate how the query builder works. You can Moderate any Eloquent Model, even User. 

###Moderate Models
You can moderate a model by referencing it's id.
```php
Post::approve($post->id);

Post::reject($post->id);
```
Or by making a query.
```php
Post::where('title', 'Horse')->approve();

Post::where('title', 'Horse')->reject();
```

###Query Models
By default only Approved models will be returned on queries. To change this behavior check the [configuration](#configuration).

#####To query the Approved Posts, run your queries as always.
```php
//it will return all Approved Posts
Post::all();

//it will return Approved Posts where title is Horse
Post::where('title', 'Horse')->get();
```
#####Query pending or rejected models.
```php
//it will return all Pending Posts
Post::pending()->get();

//it will return all Rejected Posts
Post::rejected()->get();

//it will return Approved and Pending Posts
Post::withPending()->get();

//it will return Approved and Rejected Posts
Post::withRejected()->get();
```
#####Query ALL models
```php
//it will return all Posts
Post::withAnyStatus()->get();

//it will return all Posts where title is Horse
Post::withAnyStatus()->where('title', 'Horse')->get();
```

###Model Status
To check the status of a model there are 3 helper methods which return a boolean value.
```php
//check if a model is pending
$post->isPending();

//check if a model is approved
$post->isApproved();

//check if a model is rejected
$post->isRejected();
```

##Strict Moderation
Strict Moderation means that only Approved resource will be queried. To query Pending resources along with Approved you have to disable Strict Moderation. See how you can do this in the [configuration](#configuration).

##Configuration

###Global Configuration
To configuration Moderation package globally you have to edit `config/moderation.php`.
Inside `moderation.php` you can configure the following:

1. `status_column` represents the default column 'status' in the database. 
2. `moderated_at_column` represents the default column 'moderated_at' in the database.
3. `strict` represents [*Strict Moderation*](#strict-moderation).

###Model Configuration
Inside your Model you can define some variables to overwrite **Global Settings**.

To overwrite `status` column define:
```php
const MODERATION_STATUS = 'moderation_status';
```

To overwrite `moderated_at` column define:
```php
const MODERATED_AT = 'mod_at';
```

To enable or disable [Strict Moderation](#strict-moderation):
```php
public static $strictModeration = true;
```
