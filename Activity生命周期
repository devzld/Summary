### 生命周期回调方法
1. onCreate();//创建
2. onStart();
3. onRestart();
4. onResume();//对用户可见
5. onPause(); //失去焦点，但不是完全不可见
6. onStop();  //完全不可见
7. onDestoty();//销毁

### 方法说明
+ onPause();  
//通常用于确认对持久性数据的未保存更改，停止动画，以及其他可能消耗CPU的内容；它应该迅速地执行所需操作，因为下一个Activity在它放回后才能继续执行
+ 如果系统在紧急情况下必须恢复内存，则可能不会调用onStop()和onDestroy()。
+ 系统为了恢复内存而销毁某项Activity时，系统会先调用onSaveInstanceState(Bundle outState)，然后再使Activity易于销毁。传入的Bundle可以有putString()和putInt()方法来保存Activity状态的信息，并同时传递给onCreate()和onRestoreInstanceState()。
+ Activity 类的 onSaveInstanceState() 默认实现也会恢复部分 Activity 状态，您只需为想要保存其状态的每个小部件提供一个唯一的 ID。
