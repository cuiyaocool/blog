# 问题：
看几个http的rest调用的例子：
- /users/list
- /user/{someone}/detail  

用python如何实现：
- Class.users.list生成/users/list
- Class.users(someone).detail生成/user/{someone}/detail
# 原理
## __getattr__方法
python定制类，可以实现__getattr__(self,*args, **kw)方法，当调用一个类的属性且该属性在类中未定义，会进入该方法，例子如下：
```pythonn
class Test(object):
    name = 'name1'
    
    def __getattr__(self, attr):
        if(attr == 'age'):
            return 11
        
test = Test()
print(test.name)
print(test.age)
```
该例子输出：
name1 和 11
## __call__方法
我们知道实例化一个class后，产生实例s，可以通过s.methodName()这种方式调用类中定义的方法。
在python中，可以通过实现__call__(self,*args,**kw)方法，直接调用实例，类似于s(),，此时s是一个callable对象，可以通过callable(s)进行判断，输出为True。(__python中的类为啥开头不是大写呢__)

此时对象和函数是一样的([**对象和函数可以看做一样的么?**](./Python中函数和对象是本质是一样的么.md))
```python
class Test(object):
    
    def __call__(self, something):
        print(something)
        
test = Test()
test("hello")
```
该例子输出hello
# 实践出真知，大力出奇迹
```python
class Chain1(object):
    
    def __init__(self,fun=''):
        self._path = fun
    
    def __getattr__(self, path):
        return Chain1(self._path+'/'+path)
    
    def __call__(self, detail):
        return Chain1(self._path+'/'+detail)
    
    def __str__(self):
        return (self._path)
        
    __repr__=__str__
    
    def getPath(self):
        print(self._path)
    
if __name__ == '__main__':
    print(Chain1().users('xiaoming').hello)
    print(Chain1().users.list)
    '''
    /users/xiaoming/hello
    /users/list
    '''
```
#
才疏学浅，请指正~
