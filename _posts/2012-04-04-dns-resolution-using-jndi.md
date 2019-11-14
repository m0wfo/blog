---
id: 493
title: DNS resolution using JNDI
date: 2012-04-04T21:30:55+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=493
permalink: /2012/04/04/dns-resolution-using-jndi/
categories:
  - Uncategorized
---
This is the kind of area where standing on the shoulders of JDK-giants pays dividends: the JNDI interface for looking up DNS records is surprisingly simple:

<pre><code class="language-java">import java.util.Hashtable;
import javax.naming.NamingEnumeration;
import javax.naming.NamingException;
import javax.naming.directory.Attribute;
import javax.naming.directory.Attributes;
import javax.naming.directory.DirContext;
import javax.naming.directory.InitialDirContext;

class DNSLookup {

  private static DirContext provider;

   private static void setupProvider() {
       Hashtable&lt;String,String> env = new Hashtable&lt;String, String>();
       env.put("java.naming.factory.initial", "com.sun.jndi.dns.DnsContextFactory");
       try {
         provider = new InitialDirContext(env);
       } catch (NamingException e) { System.out.println(e); }
   }

   public static String[] resolve(String domain, String record) {
       if (provider == null) setupProvider();

       try {
           Attributes query = provider.getAttributes(domain, new String[] { record });
           Attribute records = query.get(record);
           NamingEnumeration recordData = records.getAll();
           int size = records.size();
           String[] data = new String[size];
           int i = 0;
           while (i &lt; size) {
               data[i] = recordData.next().toString();
               i++;
           }
           return data;
       } catch (NamingException e) {
           System.out.println(e);
           return null;
       }
   }

}</code></pre>

Create a context, sling in a domain and record type, and off you go. Another reason why implementing [rhinode](https://github.com/m0wfo/rhinode) has involved so little work on my part... I've not looked, but I suspect the original node.js implementation is a little longer than 40 LOC<img src="https://chris.mowforth.com/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />