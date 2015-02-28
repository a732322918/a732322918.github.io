---
layout: default
title: iOS 常用代码片段
---

添加等宽视图

{% highlight objc %}

- (void)addConstraintsForViews:(NSArray *)views containerView:(UIView *)containerView
{
    for (int i = 0; i < views.count; i++) {
        UIView *view = views[i];
        NSLayoutConstraint *topConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:containerView attribute:NSLayoutAttributeTop multiplier:1.0 constant:0];
        NSLayoutConstraint *bottomConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeBottom relatedBy:NSLayoutRelationEqual toItem:containerView attribute:NSLayoutAttributeBottom multiplier:1.0 constant:-1];
        
        NSLayoutConstraint *leftConstraint;
        if (i == 0) {
            leftConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:containerView attribute:NSLayoutAttributeLeft multiplier:1.0 constant:0];
        } else {
            leftConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:views[i-1] attribute:NSLayoutAttributeRight multiplier:1.0 constant:1];
            
            NSLayoutConstraint *widthConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:views[0] attribute:NSLayoutAttributeWidth multiplier:1.0 constant:0];
            [containerView addConstraint:widthConstraint];
        }
        
        NSLayoutConstraint *rightConstraint;
        if (i == views.count - 1) {
            rightConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:containerView attribute:NSLayoutAttributeRight multiplier:1.0 constant:0];
        } else {
            rightConstraint = [NSLayoutConstraint constraintWithItem:view attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:views[i+1] attribute:NSLayoutAttributeLeft multiplier:1.0 constant:-1];
        }
        
        [containerView addConstraints:@[topConstraint, bottomConstraint, leftConstraint, rightConstraint]];
    }
}

{% endhighlight %}