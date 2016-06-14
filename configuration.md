## Application

YII_ENV
YII_DEBUG

## Frontend

# Twig Layouts

    {{ use ('hrzg/widget/widgets') }}
    {{ widget_container_widget({id: 'header'}) }}
    <div class="container">
        {{ widget_container_widget({id: 'container'}) }}
    </div>

## Backend

### Context menu items

### Extra menu items

Twig with key `extra.menuItems`

    {{ use ('hrzg/moxiecode/moxiemanager/widgets') }}
    {{ browse_button_widget( {"tagName": "a"} ) }}
    

### Commands

        'file:migrate' => [
            'class' => 'yii\console\controllers\MigrateController',
            'templateFile' => '@vendor/dmstr/yii2-db/db/mysql/templates/file-migration.php'
        ]