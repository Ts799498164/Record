1. 其实说sql呀,其实说到底就是索引,因为它不支持hash join,所以多表关联我们基本都不让用.还有就是合适的存储模型,不适合mysql的就看看是否有其它的存储解决方案

2. mysql的sql本身就弱, 像oracle里一个sql都能包含业务逻辑在里面,见别人用单个sql输出五环,五星的（是单个sql,不是存储过程）， MySQL就不可想象

