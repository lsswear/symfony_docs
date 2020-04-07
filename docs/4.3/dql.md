# 注册自定义DQL方法

Doctrine允许指定自定义DQL方法，[DQL默认方法](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/dql-user-defined-functions.html)。

自定义DQL方法：

*config/packages/doctrine.yaml*

```
doctrine:
    orm:
        dql:
            string_functions:
                test_string: App\DQL\StringFunction
                second_string: App\DQL\SecondStringFunction
            numeric_functions:
                test_numeric: App\DQL\NumericFunction
            datetime_functions:
                test_datetime: App\DQL\DatetimeFunction
```

*config/packages/doctrine.php*

```
use App\DQL\DatetimeFunction;
use App\DQL\NumericFunction;
use App\DQL\SecondStringFunction;
use App\DQL\StringFunction;

$container->loadFromExtension('doctrine', [
    'orm' => [
        // ...
        'dql' => [
            'string_functions' => [
                'test_string'   => StringFunction::class,
                'second_string' => SecondStringFunction::class,
            ],
            'numeric_functions' => [
                'test_numeric' => NumericFunction::class,
            ],
            'datetime_functions' => [
                'test_datetime' => DatetimeFunction::class,
            ],
        ],
    ],
]);
```

如果entity_managers是显示命名，若直接使用orm配置会报错：Unrecognized option "dql" under "doctrine.orm" 不可识别选项"dql"在"doctrine.orm"下

*config/packages/doctrine.yaml*

```
doctrine:
    orm:
        entity_managers:
            example_manager:
                # Place your functions here
                dql:
                    datetime_functions:
                        test_datetime: App\DQL\DatetimeFunction
```

*config/packages/doctrine.php*

```
use App\DQL\DatetimeFunction;

$container->loadFromExtension('doctrine', [
    'doctrine' => [
        'orm' => [
            'entity_managers' => [
                'example_manager' => [
                    // place your functions here
                    'dql' => [
                        'datetime_functions' => [
                            'test_datetime' => DatetimeFunction::class,
                        ],
                    ],
                ],
            ],
        ],
    ],
]);
```


