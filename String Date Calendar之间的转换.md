#String Date Calendar之间的转换

###1.Calendar 转化 String
<pre><code>
Calendar calendat = Calendar.getInstance();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
String dateStr = sdf.format(calendar.getTime());
</code></pre>

###2.String 转化Calendar
<pre><code>
String str="2012-5-27";
SimpleDateFormat sdf= new SimpleDateFormat("yyyy-MM-dd");
Date date =sdf.parse(str);
Calendar calendar = Calendar.getInstance();
calendar.setTime(date);
</code></pre>

###3.Date 转化String
<pre><code>
SimpleDateFormat sdf= new SimpleDateFormat("yyyy-MM-dd");
String dateStr=sdf.format(new Date());
</code></pre>

###4.String 转化Date
<pre><code>
String str="2012-5-27";
SimpleDateFormat sdf= new SimpleDateFormat("yyyy-MM-dd");
Date date= sdf.parse(str);
</code></pre>

###
<pre><code>
5.Date 转化Calendar
Calendar calendar = Calendar.getInstance();
calendar.setTime(new java.util.Date());
</code></pre>

###6.Calendar转化Date
<pre><code>
Calendar calendar = Calendar.getInstance();
java.util.Date date =calendar.getTime();
</code></pre>

###7.String 转成 Timestamp
<pre><code>
Timestamp ts = Timestamp.valueOf("2012-1-14 08:11:00");
</code></pre>

###8.Date 转 TimeStamp
<pre><code>
SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String time = df.format(new Date());
Timestamp ts = Timestamp.valueOf(time);
</code></pre>
