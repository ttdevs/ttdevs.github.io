
## 0x00 需求

单曲循环播放歌曲，要求过度连贯，听不出来卡顿感觉。


## 0x01 解决思路

1. MediaPlayer

    对于常见的音乐播放，我们第一时间想到的应该就是它，它有一个方法
    
    `MediaPlayer.setLooping(true);`
    
    就是用来进行单曲循环的。但是很遗憾，如果你简单的这么做，上面的目的是达不到的，会出现上一遍结束出现一个明显的停止才开始播放下一次的现象。
    
    不过最后就是用的这个组件，不过不是简单的设置 `setLooping(true)`。
    
2. SoundPool

   一段时间内可能会播放很多音乐的时候，我们首先应该选择这个。
   
3. 第三方组件

    如果没有特殊的需求，这个不是首选，特别是当引入的组件特别重的时候。    
    
因为之前踩过坑：iOS上直接播放mp3文件，单曲循环的时候播放的间隙特别长，卡顿的感觉无法接受，解决方法是将mp3转换成m4a，基本上听不出中间的过度间隙。
首先尝试了使用 `MediaPlayer` 来播放，在我的机器（MX4 Pro）上播放还勉强能接受，间隙不是非常明显，换到配置差一点的机器上就不能忍了。然后尝试了 `SoudPool` ，无论是预加载一次循环播放，还是预加载两次循环播放，中间的卡顿感觉和用 `MediaPlayer.setLooping(true);` 一样一样的。再然后，尝试macOS 下编译 `vlc for android` ，我失败了╮(╯▽╰)╭）问题总要解决的，再找其他办法。
 

## 0x02 死循环

找了很多资料，最后使用一个循环播放的方法解决了这个问题：

- 创建第一个播放器，播放；
- 同时创建第二个播放器，准备；
- 第一个播放器播放完毕立马启动第二个；
- 然后创建第三个播放器，准备；
- 如此往复，直到用户停止...

由于对 `MediaPlayer` 没有过深入的研究和使用，这个思路来一时半会自己还是想不出来的（总是会想只要创建一个播放器就够了）。这么做下来真的循环播放就没有间隙感了……

由于 `mPlayer.setLooping(true);` 是native方法，所以没有去跟具体的实现逻辑。猜测可能是重新加载或者其他原因导致单曲循环中间间隙较大（原谅我的懒，没有去拿大文件尝试）。而使用上面的方式，当播放时间大于预加载时间的时候，第一个播放器播放的时候有第二个播放器有充足的机会去完成加载然后等待播放（播放时间小于加载时间的可能性不是很大）。

``` java
private MediaPlayer mPlayer, mNextPlayer;
private int mPlayResId = R.raw.water;

public void testLoopPlayer() {
    mPlayer = MediaPlayer.create(this, mPlayResId);
    mPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
        @Override
        public void onPrepared(MediaPlayer mediaPlayer) {
            mPlayer.start();
        }
    });
    createNextMediaPlayer();
}

private void createNextMediaPlayer() {
    mNextPlayer = MediaPlayer.create(this, mPlayResId);
    mPlayer.setNextMediaPlayer(mNextPlayer);
    mPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
        @Override
        public void onCompletion(MediaPlayer mp) {
            mp.release();

            mPlayer = mNextPlayer;

            createNextMediaPlayer();
        }
    });
}
```


## 0x03 总结

这更像一个开脑洞的问题。


## 0x04 参考

- http://stackoverflow.com/questions/26274182/not-able-to-achieve-gapless-audio-looping-so-far-on-android

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


