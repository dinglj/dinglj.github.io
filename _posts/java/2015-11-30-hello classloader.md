---
layout : post
title : "hello classloader"
category : classloader
duoshuo: true
date : 2015-11-30
tags : [jdk , classloader]
SyntaxHihglighter: true
shTheme: shThemeEclipse # shThemeDefault  shThemeDjango  shThemeEclipse  shThemeEmacs  shThemeFadeToGrey  shThemeMidnight  shThemeRDark

---
>带着疑问去学习    
>1.	classloader有哪些   
2.	classloader是什么时候开始加载类    
3.	classloader加载类的过程    



JDK 默认提供了如下几种ClassLoader
---

	Bootstrp loader
	Bootstrp加载器是用C++语言写的，它是在Java虚拟机启动后初始化的，它主要负责加载%JAVA_HOME%/jre/lib,-Xbootclasspath参数指定的路径以及%JAVA_HOME%/jre/classes中的类。
	
	ExtClassLoader  
	Bootstrp loader加载ExtClassLoader,并且将ExtClassLoader的父加载器设置为Bootstrp loader.ExtClassLoader是用Java写的，具体来说就是 sun.misc.Launcher$ExtClassLoader，ExtClassLoader主要加载%JAVA_HOME%/jre/lib/ext，此路径下的所有classes目录以及java.ext.dirs系统变量指定的路径中类库。
	
	AppClassLoader 
	Bootstrp loader加载完ExtClassLoader后，就会加载AppClassLoader,并且将AppClassLoader的父加载器指定为 ExtClassLoader。AppClassLoader也是用Java写成的，它的实现类是 sun.misc.Launcher$AppClassLoader，另外我们知道ClassLoader中有个getSystemClassLoader方法,此方法返回的正是AppclassLoader.AppClassLoader主要负责加载classpath所指定的位置的类或者是jar文档，它也是Java程序默认的类加载器。

---
	双亲委托模型
---

	Java中ClassLoader的加载采用了双亲委托机制，采用双亲委托机制加载类的时候采用如下的几个步骤：
	
	当前ClassLoader首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。
	
	每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，等下次加载的时候就可以直接返回了。
	
	当前classLoader的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到bootstrp ClassLoader.
	
	当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。
	
	说到这里大家可能会想，Java为什么要采用这样的委托机制？理解这个问题，我们引入另外一个关于Classloader的概念“命名空间”， 它是指要确定某一个类，需要类的全限定名以及加载此类的ClassLoader来共同确定。也就是说即使两个类的全限定名是相同的，但是因为不同的 ClassLoader加载了此类，那么在JVM中它是不同的类。明白了命名空间以后，我们再来看看委托模型。采用了委托模型以后加大了不同的 ClassLoader的交互能力，比如上面说的，我们JDK本生提供的类库，比如hashmap,linkedlist等等，这些类由bootstrp 类加载器加载了以后，无论你程序中有多少个类加载器，那么这些类其实都是可以共享的，这样就避免了不同的类加载器加载了同样名字的不同类以后造成混乱。

--


{% highlight java linenos %}
protected final Class<?> findLoadedClass(String name) {
        if (!checkName(name))
            return null;
        return findLoadedClass0(name);
    }
{% endhighlight %}



{% highlight java linenos %}
 protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
			//查找当前classloader下class
            Class c = findLoadedClass(name);
            if (c == null) {
                try {
					//查找父类classloader中class 递归
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
						//递归直到父类classloader为null,查找Bootstrploader中的class
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
    }
{% endhighlight %}


