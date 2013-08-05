---
layout: post
title:  "簡単にナビゲーションバーのバックボタンをカスタマイズする方法"
date:   2013-08-04 12:31:21
categories: diary
---

## バックボタン
バックボタンは他の UIBarButtonItem と違って、カスタマイズが少し大変です。
何故かというと、他のとは違って UINavigationController pushViewController:animated: された UIViewController にはデフォルトでバックボタンが付いてしまうからです。
では、どのようにしたら簡単にカスタマイズできるかみてみましょう。

## 実装
全体的な方針としては、 UINavigationController と UIViewController をそれぞれサブクラス化します。
以後、アプリ内で UIViewController や UINavigationController をサブクラス化する時はそのサブクラスを使う、いわばアプリ内基底クラスを作ります。

では、二つの基底クラスの実装をみていきます。まずは UIViewController から。

## UIViewController
全ての UIViewController の基底クラスは UINavigationControllerDelegate プロトコルに準拠するようにします。
勘の良い方は分かると思いますが delegate を渡す部分を UINavigationController の基底クラスで実装します。
肝心のバックボタンをセットする実装は以下の UINavigationControllerDelegate の delegate method 内で行います。

{% highlight objective-c %}
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{

    if ( [navigationController.viewControllers count] > 1 ) {
        viewController.navigationItem.leftBarButtonItem = [self backButton];
    }
}
{% endhighlight %}

いかがですか？簡単ですよね。上記の delegate method は UIViewController を表示する直前に呼ばれます。
backButton には UIBarButtonItem を initWithCustomView して作成するイメージです。
## UINavigationController
UINavigationController の基底クラスの実装は簡単です。
もし UIViewController の基底クラスに delegate を渡す必要が無いのならば、上記の UIViewController の delegate 内の実装をこちらに寄せても良いと思います。
なぜなら UINavigationController では UIViewController に delegate を渡すだけだからです。
以下がそのコードになります。

{% highlight objective-c %}
- (id)initWithRootViewController:(UIViewController *)rootViewController
{
        self = [super initWithRootViewController:rootViewController];
        if ( self ) {
             self.navigationItem.hidesBackButton = YES;
             if ( [rootViewController isKindOfClass:[YourCustomizedViewController class]] ) {
                 rootViewController.navigationController.delegate = (YourCustomizedViewController *)rootViewController;
             }
        }
        return self;
}

- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    if ( !viewController ) {
        return;
    }
    if ( [rootViewController isKindOfClass:[YourCustomizedViewController class]] ) {
        rootViewController.navigationController.delegate = (YourCustomizedViewController *)rootViewController;
    }
    [super pushViewController:viewController animated:animated];
}
{% endhighlight %}

簡単ですね。 UINavigationController にわたってくる UIViewController 全てに delegate を渡しています。
