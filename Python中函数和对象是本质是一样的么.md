在文章[神奇的python链式调用](./神奇的python链式调用.md)中，提到一个疑惑，**函数和对象一样么**

### 思考  
1. 一样的话说明函数也有属性，那么函数真的有么  
2. 函数和对象一样可以动态添加属性么  
3. 函数也是类的实例么
4. 对象也可以调用么

#
```python
def testFunc():
    pass

def testSubFunc():
    print('sub func!')

if __name__ == '__main__':
    # 动态加入属性
    testFunc.someAttr = 11
    print( testFunc.someAttr )
    # 动态加入方法
    testFunc.subFunc = testSubFunc
    # 调用动态加入的方法
    testFunc.subFunc()
    print( dir(testFunc) )
    print( type(testFunc) )

```
该例子中，我们定义了一个函数testFunc,如果函数和对象一样，那么第一点，他有属性，他还可以动态加入属性，包括静态属性和方法，于是我们进行了验证，输出如下：
```python
11
sub func!
['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'someAttr', 'subFunc']
<class 'function'>
```
由此可以看出，函数也是一个对象，他本身有很多的属性和方法，还能动态加入属性和方法，他是类function的一个实例。
#
那么对象也可以调用么?是的，类对象实现__call__方法后是可以直接调用的。
```python
class testclass(object):
    def __call__(self):
        print('对象调用')
def testSubFunc():
    print('sub Func!')

if __name__ == '__main__':
    # 动态加入属性
    testclass.someAttr = 11
    print( testclass.someAttr )
    # 动态加入方法
    testclass.subFunc = testSubFunc
    # 调用动态加入的方法
    testclass.subFunc()
    print( dir(testclass) )
    print( type(testclass) )
    
    instance = testclass()
    instance()
    print(callable(testclass))
```
如上，定义类是重写了__call__方法，于是实例可以直接调用，当然call可以加入处理self以外的参数。  
#
**另外：** 这和直接调用类不一样，直接调用类是创建实例，相关方法是init。也**不能**先实例化对象后，给对象添加call方法   
#
可以直接调用的是一个callable对象，可以使用 callable(testclass)进行判断，可以直接调用的对象返回true，当然，函数返回的是true。
