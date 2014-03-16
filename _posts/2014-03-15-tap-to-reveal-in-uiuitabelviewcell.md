---
layout: post
title: "Tap to Reveal in UIUITabelViewCell" 
---
One of the most versatile UIViews in iOS SDK is the `UITableView`. It is really useful to display a list of items. If a detailed view of the item is required, a details `UIView` is created and is pushed on top of the exiting view. This works most of the time. On certain occasions the item being in a position is important and pushing a new view results in a loss of context. So I wanted to figure out if the height of the `UITableViewCell` can be changed based on its content. This would achieve a **Tap to Reveal** animation as shown below.


![image](https://raw.github.com/malolans/HTARevealOptions/master/ScreenCap.gif)

A quick and easy way to perform this operation would be to change the height of the selected `UITableViewCell`. The selected index is stored as a property of the `UITableViewController`. If a new cell is tapped, the property is updates and the height adjusted accordingly.

Now to make this work, the height of the `UITableViewCell` can be specified in the `UITableViewDelegate` method `heightForRowAtIndexPath`.

{% highlight objective-c %}
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if ([indexPath isEqual:selectedIndex]) {
        return self.tableView.rowHeight * 2;
    } else {
        return self.tableView.rowHeight;
    }
}
{% endhighlight %}

This is the first step in getting the height of the `UITableViewCell` to change. The next step is to force the redraw operation on the tapped cell. This can be done by using the `UITableView` method `reloadRowsAtIndexPaths`. The reload is done when one of the rows is tapped. Along with this, in the `didSelectRowAtIndexPath` methods, the handling of the selected Index Path is also performed. The reload operation is performed with `UITableViewRowAnimationFade` type animation. This makes the hidden sections slide up or down.

{% highlight objective-c %}
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    HTAData *aCharacter = [characterData objectAtIndex:indexPath.row];
    if (![indexPath isEqual:selectedIndex]) {
        selectedIndex = indexPath;
        aCharacter.isSelected = YES;
    } else {
        selectedIndex = nil;
        aCharacter.isSelected = NO;
    }

    [tableView reloadRowsAtIndexPaths:[NSArray arrayWithObject:indexPath]
                     withRowAnimation:UITableViewRowAnimationFade];
}
{% endhighlight %}

A similar operation is performed when the current active cell is deselected.

{% highlight objective-c %}
-(void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (![indexPath isEqual:selectedIndex]) {
        selectedIndex = nil;
        HTAData *aCharacter = [characterData objectAtIndex:indexPath.row];
        aCharacter.isSelected = NO;
    }
    [tableView reloadRowsAtIndexPaths:[NSArray arrayWithObject:indexPath]
                     withRowAnimation:UITableViewRowAnimationFade];
}
{% endhighlight %}

Now the `UITableViewCell` handles the hiding and showing of the sections below the default height based on the selected state of the cell. It also handles the movement of the our custom `UITableViewCellSeparator`.

{% highlight objective-c %}
- (void)layoutSubviews
{
    [super layoutSubviews];
    CGRect bottomBarPostion;
        
    self.nameLabel.frame = CGRectMake(15, 0, 290, 44);
    self.detialsLabel.frame = self.nameLabel.frame;
    
    if (!self.cellData.isSelected) {
        self.fullNameView.hidden = YES;
        bottomBarPostion = CGRectMake(0.0, self.nameView.bounds.size.height - 0.5, self.bounds.size.width, 0.5);
    } else {
        bottomBarPostion = CGRectMake(0.0, self.bounds.size.height - 0.5, self.bounds.size.width, 0.5);
        self.fullNameView.hidden = NO;
    }
    
    self.bottomBar.frame = bottomBarPostion;
}
{% endhighlight %}

Now this is all setup to hide and show the extra details and still maintain the context in the `UITableView`.

I have uploaded the project on [github](https://github.com/malolans/HTARevealOptions). Please let me know if you have any comments.