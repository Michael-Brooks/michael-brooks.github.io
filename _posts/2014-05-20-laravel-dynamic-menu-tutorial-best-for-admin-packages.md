---
layout: post
title: Laravel Dynamic Menu Tutorial (Best for Admin packages)
---

[Learn more Laravel](https://michaelbrooks.co.uk/post/top-5-laravel-books)

<!-- Begin MailChimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; }
	/* Add your own MailChimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="//michaelbrooks.us2.list-manage.com/subscribe/post?u=76e6479bd0342239b07c0da4e&id=5b49e965c6" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Sign up to recieve an updated tutorial</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_76e6479bd0342239b07c0da4e_5b49e965c6" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->

Thanks to this very Laravel topic over at [Laravel.io](http://laravel.io) and [Josh Benham](https://github.com/joshbenham) I managed to find out how to make my very own dynamic menu (which is great if you would like to make a reusable admin package). In this tutorial, I will show you exactly how it's done and hopefully you will now be able to make amazing admin packages for Laravel.

For those that just want the package then you can download it the [Laravel dynamic navigation package](https://michaelbrooks.co.uk/post/laravel-dynamic-navigation-package). Otherwise, continue below.

First off, I am doing this in my very own package, so my directory structure will look pretty much like anyone elses file structure which will be within the src/Package/Admin. Hopefully you should be able to follow this along just fine, but if you have any problems or even any suggestions on improvements then feel free to let me know.

Within my Admin folder, I create a new folder called services and create a file called 'Menu.php', this is the code you should type within this file...
{% highlight php %}
<?php

namespace Package\Admin\Services;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\HTML;
use Illuminate\Support\Facades\Request;
use Illuminate\Support\Facades\URL;

class Menu {
    
    protected $items;
    protected $current;
    protected $currentKey;

    public function __construct() {
        $this->current = Request::url();
    }
        
    /*
     * Shortcut method for create a menu with a callback.
     * This will allow you to do things like fire an even on creation.
     * 
     * @param callable $callback Callback to use after the menu creation
     * @return object
     */
    public static function create($callback) {
        $menu = new Menu();
        $callback($menu);
        $menu->sortItems();

        return $menu;
    }
        
    /*
     * Add a menu item to the item stack
     * 
     * @param string $key Dot separated hierarchy
     * @param string $name Text for the anchor
     * @param string $url URL for the anchor
     * @param integer $sort Sorting index for the items
     * @param string $icon URL to use for the icon
    */
    public function add($key, $name, $url, $sort = 0, $icon = null)
    {
        $item = array(
            'key'       => $key,
            'name'      => $name,
            'url'       => $url,
            'sort'      => $sort,
            'icon'      => $icon,
            'children'  => array()
        );

        $children = str_replace('.', '.children.', $key);
        array_set($this->items, $children, $item);

        if($url == $this->current) {
            $this->currentKey = $key;
        }
    }
        
    /*
     * Recursive function to loop through items and create a menu
     * 
     * @param array $items List of items that need to be rendered
     * @param boolean $level Which level you are currently rendering
     * @return string
    */
    public function render($items = null, $level = 1)
    {
        $items = $items ?: $this->items;

        $attr = array(
            'class' => 1 === $level ? 'menu level-1' : 'level-' . $level
        );

        $menu = '<ul' . HTML::attributes($attr) . '>';

        foreach($items as $item) {
            $classes = array('menu__item');
            $classes[] = $this->getActive($item);
    
            $has_children = sizeof($item['children']);
    
            if ($has_children) {
                $classes[] = 'parent';
            }
    
            $menu .= '<li' . HTML::attributes(array('class' => implode(' ', $classes))) . '>';
            $menu .= $this->createAnchor($item);
            $menu .= ($has_children) ? $this->render($item['children'], ++$level) : '';
            $menu .= '</li>';
        }

        $menu .= '</ul>';

        return $menu;
    }
        
    /*
     * Method to render an anchor
     * 
     * @param array $item Item that needs to be turned into a link
     * @return string
     */
    private function createAnchor($item)
    {
        $output = '<a class="menu__item" href="' . $item['url'] . '">';
        $output .= $this->createIcon($item);
        $output .= $item['name'];
        $output .= '</a>';

        return $output;
    }
        
    /*
     * Method to render an icon
     * 
     * @param array $item Item that needs to be turned into a icon
     * @return string
    */
    private function createIcon($item)
    {
        $output = '';
    
        if($item['icon']) {
            $output .= sprintf(
                '<i class="fa fa-%s"></i> ',
                $item['icon']
            );
        }
            
        return $output;
    }
        
    /*
     * Method to sort through the menu items and put them in order
     * 
     * @return void
    */
    private function sortItems() {
        usort($this->items, function($a, $b) {
            if($a['sort'] == $b['sort']) {
                return 0;
            }
                
            return ($a['sort'] < $b['sort'] ? -1 : 1);
        });
    }
        
    /*
     * Method to find the active links
     * 
     * @param array $item Item that needs to be checked if active
     * @return string
    */
    private function getActive($item)
    {
        $url = trim($item['url'], '/');
    
        if ($this->current === $url)
        {
            return 'active current';
        }
    
        if(strpos($this->currentKey, $item['key']) === 0) {
            return 'active';
        }
    }
        
}
{% endhighlight %}
This code will set you up for all the behind the scenes stuff. Hopefully most of it is pretty self-explanatry (like most of Laravel) and should be well commented to give you a grasp on what is going on.

Next, we will make our navigation which will be within the views folder, I created an inc folder within my views to keep things tidy called 'navigation.blade.php'. And my code goes a little like this...
```PHP
<?php
use Package\Admin\Services\Menu;
?>

<nav>
{% raw %}
{{
    Menu::create(function($menu) {
        Event::fire('admin.menu.build', $menu);
    })->render();
}}
{% endraw %}
</nav>
```
This will display your navigation, but the only trouble is that we haven't actually built it. Here we have the code to build our dynamic navigation menu. Inside your AdminServiceProvider.php you need to add this line into your public function boot()
```PHP
include __DIR__.'/../../events.php';
```
This will tell your package where to find the events page which will build up your navigation (your events.php needs to be in the same place as your routes and filters, which should be in the src folder)...
{% highlight php %}
<?php
Event::listen('admin.menu.build', function($menu){
    $menu->add('index', 'Index', URL::route('Admin::index.index'), 1, 'dashboard');
    $menu->add('users', 'Users', URL::route('Admin::users.index'), 100, 'users');
});
{% endhighlight %}
Here we use the event listen method used in Laravel to build our menu, the first part of our add method gives a key which enables a check to see if it's the current page or not. The second part adds text to your link, URL::route will be the route name which you use in your routes.php. We then have these numbers which says where they should be displayed on the nav and users or dashboard is a font aswesome icon.

Hopefully this is enough to get anyone started on a dynamic nav, if you need to use it on more packages then just add the code to the service provider, add another events.php and then create similar code to that above which will build up your navigation even more.

I would also like to thank [Josh Benham](https://github.com/joshbenham) for his help and support. Check out the discussion at [laravel.io](http://laravel.io/forum/04-16-2014-dynamic-menu-system?page=1#reply-5576).

If you would like to pull in the package I created then take a look at my next post [Laravel dynamic navigation package](https://michaelbrooks.co.uk/post/laravel-dynamic-navigation-package).

Feel free to add your own feedback in the comments below

[Learn more Laravel](https://michaelbrooks.co.uk/post/top-5-laravel-books)

---

