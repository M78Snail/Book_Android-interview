#### Touch事件分析

> 事件分发：public boolean dispatchTouchEvent\(MotionEvent ev\)

Touch事件发生时Activity的 **dispatchTouchEvent\(MotionEvent ev\) **方法会以隧道方式（从根元素依次往下传递直到最内层子元素或在中间某一元素中由于某一条件停止传递）将事件传递给最外层View的 **dispatchTouchEvent\(MotionEvent ev\) **方法，并由该View的**dispatchTouchEvent\(MotionEvent ev\) **方法对事件进行分发。**dispatchTouchEvent **的事件分发逻辑如下：

* 如果 **return true ,**事件会分发给当前View并由**dispatchTouchEvent\(MotionEvent ev\)**方法进行消费**，**同时事件会停止向下传递
* 如果 **return false**，事件会分发给父View的 **onTouchEvent **进行消费
* 如果返回系统默认的**super.dispatchTouchEvent\(ev\)**,事件会自动的分发给当前View的**onIntercepTouchEvent**方法。

> 事件拦截：public boolean onIntercepTouchEvent\(MotionEvent ev\)

在外层View的**dispatchTouchEvent\(MotionEvent ev\)**方法返回系统默认的**super.dispatchTouchEvent\(MotionEvent ev\)** 情况下，事件会自动的分发给当前View的**onIntercepTouchEvent**方法。**onIntercepTouchEvent **的事件拦截逻辑如下：

* 如果 **return true **,表示会进行拦截，并交给当前View的onTouchEvent进行处理
* 如果 **return false，**表示会进行放行，当前View上的事件会传递到子View上，再由子View的dispatchTouchEvent来开始这个事件的分发
* 如果返回系统默认的**super.onIntercepTouchEvent\(ev\)**,默认表示不拦截该事件，并将事件传递给下一层View的dispatchTouchEvent。

> 事件响应：public boolean onTouchEvent\(MotionEvent ev\)

在dispatchTouchEvent返回**super.dispatchTouchEvent\(ev\)**并且**onIntercepTouchEvent**返回 true 或返回**super.onIntercepTouchEvent**的情况下onTouchEvent会被调用。onTouchEvent** **的事件响应逻辑如下：

* 如果 **return true **,表示会进行接收并处理事件
* 如果 **return false，**表示这个事件会从当前View向上传递并且都是由上层View的onTouchEvent来接收，如果上层onTouchEvent也返回false，这个事件就会消失，并且接收不到下一次事件
* 如果返回系统默认的**super.onTouchEvent\(ev\)**默认处理事件的逻辑和返回false时相同

---

