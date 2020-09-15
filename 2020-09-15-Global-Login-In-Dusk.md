---
title: 在 dusk 中使用全局登录
date: 2020-09-11 19:37:02
tags: PHPUnit,Dusk
categories: PHPUnit
---

调整 DuskTestCase，增加以下方法。

```php
    /**
     * @var bool
     */
    protected $login = true;

    public function login(Browser $browser)
    {
        // $browser->loginAs($this->getUser(), 'super');
    }


    /**
     * @param \Facebook\WebDriver\Remote\RemoteWebDriver $driver
     *
     * @return \Laravel\Dusk\Browser
     */
    protected function newBrowser($driver)
    {
        $browser = (new Browser($driver));
        $browser->setActionCollector(new BrowserActionCollector($this->getTestName()));
        $browser->resolver->prefix = 'html';

        $this->login($browser);

        return $browser;
    }

    /**
     * @return Administrator
     */
    protected function getUser()
    {
        if ($this->user) {
            return $this->user;
        }

        $admin = Administrator::orderBy('id', 'asc')->first() ?? factory(Administrator::class)->create();

        return $this->user = $admin;
    }
```