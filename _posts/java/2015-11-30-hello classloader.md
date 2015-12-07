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


---
{% highlight java linenos %}
protected final Class<?> findLoadedClass(String name) {
        if (!checkName(name))
            return null;
        return findLoadedClass0(name);
    }
{% endhighlight %}
---


---
{% highlight java linenos %}
 protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
			//查找当前classloader下class
            Class c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
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

---
