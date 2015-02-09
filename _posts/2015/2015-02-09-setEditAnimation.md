---
layout: page
categories:
- blog
title: UITableView - setEditでセル追加アニメーションを正しく実行させる
---

いやぁ，長らく触ってないとめちゃくちゃなコードを書いてしまいますね．
UITableViewでeditモードに移行したタイミングで「追加のためのセル」を表示するコードとか，すっかり適当に書いてました．
ダメなコードはこんな感じ．

    - (void)setEditing:(BOOL)editing animated:(BOOL)animated {
        [super setEditing:editing animated:animated];
        [self.tableView reloadData];
    }

よりだめになると，````reloadData````を時間差で読んだりしますね．くずコードです．

正しくは，

    - (void)setEditing:(BOOL)editing animated:(BOOL)animated {
        [super setEditing:editing animated:animated];
        
        NSIndexPath *addCellIndexPath = [NSIndexPath indexPathForRow:self.items.count inSection:0];
        
        [self.tableView beginUpdates];
        
        if (self.editing)
            [self.tableView insertRowsAtIndexPaths:@[addCellIndexPath] withRowAnimation:UITableViewRowAnimationTop];
        else
            [self.tableView deleteRowsAtIndexPaths:@[addCellIndexPath] withRowAnimation:UITableViewRowAnimationTop];
        
        [self.tableView endUpdates];
    }

です．

````self.editing````フラグに基づいて，セルの数等を````dataSource````デリゲートで調節することを忘れずに．
がんばります．