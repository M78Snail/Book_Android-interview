从ViewRootImpl类的**performTraversals\(\)**方法开始，通过判断选择：

> performMeasure\(\)--&gt;measure\(\)--&gt;onMeasure\(\)  
> performLayout\(\)--&gt;layout\(\)--&gt;onLayout\(\)  
> performDraw\(\)--&gt;draw\(\)--&gt;onDraw\(\)

**onMeasure\(\):**

> specMode:  
> 1. EXACTLY:match-parent,固定dp  
> 2. AT\_MOST:wrap\_content



