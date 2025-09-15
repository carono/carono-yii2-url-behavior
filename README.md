# Yii2 URL Behavior

A flexible behavior for Yii2 that provides dynamic URL generation based on user roles, application context, and custom rules.

## Features

- Role-based URL generation
- Multi-application support
- Flexible rule configuration
- Caching support for performance optimization
- Customizable URL rule parameters

## Installation

Add the package to your `composer.json`:
```json
{
    "require": {
        "carono/yii2-url-behavior": "*"
    }
}
```

## Usage

### Attaching the Behavior

Attach the behavior to your ActiveRecord model:

```php
use carono\yii2\behaviors\UrlBehavior;

class Post extends \yii\db\ActiveRecord
{
    public function behaviors()
    {
        return [
            [
                'class' => UrlBehavior::class,
                'rules' => [
                    ['view', 'url' => ['post/view', 'id' => 'id']],
                    ['update', 'url' => ['post/update', 'id' => 'id'], 'role' => 'admin'],
                    ['delete', 'url' => ['post/delete', 'id' => 'id'], 'role' => ['admin', 'moderator']],
                ],
                'defaultUrl' => ['site/index'],
                'functionAlias' => 'getUrl' // Default method name
            ]
        ];
    }
}
```

### Configuration Options

- `rules`: Array of URL rules or a string method name that returns rules
- `defaultUrl`: Default URL when no rules match
- `functionAlias`: Method name to expose (default: `getUrl`)
- `authManager`: Auth manager component ID (default: `authManager`)
- `ruleClass`: Custom rule class implementation

### Rule Configuration

Each rule can have these properties:

- `action`: Action name (required)
- `url`: URL pattern or callable (required)
- `role`: Role or array of roles (optional)
- `application`: Application ID restriction (optional)
- `params`: Additional parameters (optional)

### URL Generation

```php
$post = Post::findOne(1);

// Get URL as array
$urlArray = $post->getUrl('view');

// Get absolute URL string
$urlString = $post->getUrl('view', true);
```

### Advanced Rule Examples

```php
'rules' => [
    [
        'view', 
        'url' => ['post/view', 'id' => 'id'],
        'role' => 'user',
        'application' => 'frontend'
    ],
    [
        'update',
        'url' => function($model) {
            return ['post/update', 'id' => $model->id, 'slug' => $model->slug];
        },
        'role' => ['admin', 'editor']
    ],
    [
        'delete',
        'url' => ['post/delete'],
        'params' => [
            'id' => 'id',
            'timestamp' => function($model) {
                return time();
            }
        ]
    ]
]
```

## Rule Class Reference

The `UrlRule` class provides these properties:

- `action`: Target action name
- `url`: URL pattern or callable
- `role`: Role(s) required
- `application`: Application restriction
- `params`: Parameter configuration
- `authManager`: Auth manager component
- `cache`: Cache component for role caching
- `model`: Related model instance

## Caching

The behavior supports role caching to improve performance:

```php
'rules' => [
    ['view', 'url' => ['post/view'], 'role' => 'user', 'duration' => 3600]
]
```

## Advanced Usage

### Custom Rule Class

Create custom rule class by extending `UrlRule`:

```php
class CustomUrlRule extends UrlRule
{
    public function compare($action, $user)
    {
        // Custom logic here
        return parent::compare($action, $user);
    }
}
```

### Dynamic Rules

Define rules using a model method:

```php
public function behaviors()
{
    return [
        [
            'class' => UrlBehavior::class,
            'rules' => 'getUrlRules'
        ]
    ];
}

public function getUrlRules()
{
    return [
        ['view', 'url' => ['post/view', 'id' => 'id']],
        // ... more rules
    ];
}
```

## License

MIT