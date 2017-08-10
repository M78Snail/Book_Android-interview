锁定屏幕只会调用**onPause\(\),**开屏调用**onResume\(\)**

异常终止：



> 保存数据:onSaveInstanceState\(\)方法{onStop之前，但不一定是在onPause\(\)之后或之前}

> 恢复数据:onRestoreInstanceState\(\)方法{在onStart\(\)之后}



