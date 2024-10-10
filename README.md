```
loginAction.java
package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionServlet;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;
import fjs.cs.dao.loginDao;
import fjs.cs.form.loginForm;

public class loginAction extends Action {

    private loginDao loginDao;

    @Override
    public void setServlet(ActionServlet servlet) {
        super.setServlet(servlet);
        WebApplicationContext context = WebApplicationContextUtils.getRequiredWebApplicationContext(servlet.getServletContext());
        this.loginDao = (loginDao) context.getBean("loginDao");
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        loginForm loginForm = (loginForm) form;

        // Lấy thông tin người dùng từ form
        String userName = loginForm.getUserID();
        String passWord = loginForm.getPassWord();

        // Kiểm tra thông tin đăng nhập
        int count = loginDao.checkLogin(userName, passWord);

        if (count > 0) {
            // Đăng nhập thành công
            return mapping.findForward("success");
        } else {
            // Đăng nhập thất bại
            request.setAttribute("errorMessage", "Invalid username or password.");
            return mapping.findForward("failure");
        }
    }
}
```
```
loginDao
package fjs.cs.dao;

import java.util.List;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;


public class loginDao {
	

	private SessionFactory sessionFactory;


	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}

    public int checkLogin(String userName, String passWord) {
        Session session = sessionFactory.openSession();
        System.out.println(userName);
        String hql = "SELECT COUNT(*) FROM Login l WHERE l.deleteYMD IS NULL AND l.userID = :userID AND l.passWord = :passWord";
        
        Long count = (Long) session.createQuery(hql)
                                  .setParameter("userID", userName)
                                  .setParameter("passWord", passWord)
                                  .uniqueResult();
        session.close();
        System.out.println(count.intValue());
        return count.intValue();
    }
}
```
```
loginForm.java
package fjs.cs.form;

import org.apache.struts.action.ActionForm;

public class loginForm extends ActionForm {
	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;
	private String userID; 
	private String passWord;
	
	public loginForm() {
		super();
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassWord() {
		return passWord;
	}

	public void setPassWord(String passWord) {
		this.passWord = passWord;
	} 
}

```
```
Login.java (model)
package fjs.cs.model;

public class Login {
	private int psn_cd; 
	private String userID;
	private String userName;
	private String passWord;
	private String deleteYMD;
	public int getPsn_cd() {
		return psn_cd;
	}
	public void setPsn_cd(int psn_cd) {
		this.psn_cd = psn_cd;
	}
	public String getUserID() {
		return userID;
	}
	public void setUserID(String userID) {
		this.userID = userID;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public String getPassWord() {
		return passWord;
	}
	public void setPassWord(String passWord) {
		this.passWord = passWord;
	}
	public String getDeleteYMD() {
		return deleteYMD;
	}
	public void setDeleteYMD(String deleteYMD) {
		this.deleteYMD = deleteYMD;
	}
	
	
}
```
```
Login.hbm.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <class name="fjs.cs.model.Login" table="MSTUSERS">
        <id name="psn_cd" column="PSN_CD" type="int">
        </id>
        <property name="userID" column="USERID" type="string" length="50"/>
        <property name="userName" column="USERNAME" type="string" length="100"/>
        <property name="passWord" column="PASSWORD" type="string" length="100"/>
        <property name="deleteYMD" column="DELETE_YMD" type="timestamp"/>
    </class>
</hibernate-mapping>
```
```
hibernate.cfg.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/CustomerSystem</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">0973129264a</property>

        <!-- JDBC connection pool settings -->
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- Specify dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="hibernate.show_sql">true</property>

        <!-- Drop and re-create the database schema on startup -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Names the annotated entity class -->
        <mapping resource="fjs/cs/model/Login.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
```
login.jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %> 
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style type="text/css">
	body {
		margin-left :20px;
		margin-right : 20px;
		background-color : #87CEFA;	
	}
	.header {
		color : red;
		border-bottom : 2px solid;
		
	}
    .login-container {
        display: flex;
        justify-content: center;
    }

    .login-form {
        display: flex;
        flex-direction: column;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        width: 300px; /* Set a width for the form */
    }

    .form-title {
        font-size: 24px;
        margin-bottom: 20px;
        text-align: center;
        color : blue;
    }

    .form-group {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 15px;
    }

    .form-group label {
        margin-right: 10px;
        width: 80px; /* Set a fixed width for labels */
    }

    .form-group input {
        width: 70%;
        padding: 5px;
        border: 1px solid #ccc;
        border-radius: 2px;
    }

    .login-form input[type="submit"],
    .login-form input[type="button"] {
        width: 120px;
        margin: 5px;
        cursor: pointer;
    }
	
    #errorMessageDiv {
        margin-bottom: 15px;
        color: red;
        text-align: center;
        min-height: 20px; /* de form khong bi day xuong khi hien ra  */
    }
    .btnDiv {
        display: flex;
        justify-content: center;
    }
</style>
<title>Login</title>
</head>
<body>
	<div class= header>
		<h1>TRAINING</h1>
	</div>				
	<div class="container">
		<div class="login-container">
		    <div class="login-form">
		        <div class="form-title">Login</div> 
		        <div id="errorMessageDiv">
		            <html:errors />
		        </div>
		        <html:form action="/login" method="post">
		            <div class="form-group">
		                <label for="username">Username:</label>
		                <html:text property="userID" />
		            </div>
		            <div class="form-group">
		                <label for="passWord">Password:</label>
		                <html:password property="passWord" />
		            </div>
		            <div class= "btnDiv">
		            	<html:submit value= "Login" property= "action"/>
		            	<html:submit value="Clear" property="action" onclick="clearForm()" />
		            </div>
		        </html:form>
		    </div>
		</div>
	</div>
</body>
</html>
```
```
applicationContext
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình datasource kết nối MySQL -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/Customersystem"/>
        <property name="username" value="root" />
        <property name="password" value="0973129264a" />
    </bean>

    <bean id="sessionFactory"
        class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />
    </bean>
    <bean id="loginDao" class="fjs.cs.dao.loginDao">
    	<property name="sessionFactory" ref="sessionFactory"/>
	</bean>

    
</beans>
```
```
struts-config
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts-config PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN"
    "dtd/struts-config_1_3.dtd">

<struts-config>
	<form-beans>
	    <form-bean name="loginForm" type="fjs.cs.form.loginForm"/>
	</form-beans>
	
	<action-mappings>
	    <action path="/login"
	            type="fjs.cs.action.loginAction"
	            name="loginForm"
	            scope="request"
	            validate="true">
	        <forward name="success" path="/search.do" redirect="true"/>
	        <forward name="failure" path="/jsp/login.jsp" />
	    </action>
	    
	</action-mappings>
	<message-resources parameter="message" />
</struts-config>
```
```
web.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC 
    "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" 
    "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
    <display-name>HelloStruts1x</display-name>

    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
    </context-param>
    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Struts Action Servlet -->
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```








-------------------------TOI DAY LA HETTTT ---------------------------------------------------------
























```
hibernate.cfg.xml
Đây là tệp cấu hình Hibernate để kết nối với SQL Server.
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</property>
        <property name="hibernate.connection.driver_class">com.microsoft.sqlserver.jdbc.SQLServerDriver</property>
        <property name="hibernate.connection.url">jdbc:sqlserver://localhost:1433;databaseName=CustomerSystem</property>
        <property name="hibernate.connection.username">CustomerSystem</property>
        <property name="hibernate.connection.password">123456</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="show_sql">true</property>
        
        <mapping resource="fjs/cs/model/MstUser.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
```
loginDao.java
Tệp này đã được cập nhật để sử dụng Hibernate thay vì JDBC.
package fjs.cs.dao;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;
import fjs.cs.model.MstUser;
import fjs.cs.util.HibernateUtil;

public class loginDao {
    public int checkLogin(String userName, String passWord) {
        int cnt = 0;
        Transaction transaction = null;

        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            transaction = session.beginTransaction();

            String hql = "SELECT COUNT(*) FROM MstUser WHERE deleteYmd IS NULL AND userID = :username AND password = :password";
            Query<Long> query = session.createQuery(hql, Long.class);
            query.setParameter("username", userName);
            query.setParameter("password", passWord);

            cnt = query.uniqueResult().intValue();

            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        }
        return cnt;
    }
}

```
```
loginAction.java
package fjs.cs.action;

import java.sql.SQLException;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import fjs.cs.dao.loginDao;
import fjs.cs.form.loginForm;

public class loginAction extends Action {
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest req, HttpServletResponse resp) {
        loginForm loginForm = (loginForm) form;
        String userid = loginForm.getUserID();
        String password = loginForm.getPassWord();
        String action = req.getParameter("action");

        ActionMessages errors = new ActionMessages();
        if ("Login".equals(action)) {
            if (userid == null || userid.trim().isEmpty()) {
                errors.add("username", new ActionMessage("error.username.required"));
                saveErrors(req, errors);
                return mapping.findForward("failure");
            }
            if (password == null || password.trim().isEmpty()) {
                errors.add("password", new ActionMessage("error.password.required"));
                saveErrors(req, errors);
                return mapping.findForward("failure");
            } else {
                loginDao loginDao = new loginDao();
                int cnt = loginDao.checkLogin(userid, password);
                if (cnt == 1) {
                    HttpSession session = req.getSession();
                    session.setAttribute("userid", userid);
                    return mapping.findForward("success");
                } else {
                    errors.add("login", new ActionMessage("error.login.invalid"));
                    saveErrors(req, errors);
                    return mapping.findForward("failure");
                }
            }
        }
        if ("Clear".equals(action)) {
            req.setAttribute("errorMessageDiv", "");
            loginForm.setUserID("");
            loginForm.setPassWord("");
            return mapping.findForward("failure");
        }
        return mapping.findForward("failure");
    }
}

```
```
loginDto.java
Tệp này có thể không cần thay đổi, vì nó chỉ là DTO. Dưới đây là mã nguồn cho bạn:
package fjs.cs.dto;

public class loginDto {
    private String userID; 
    private String passWord;

    public loginDto(String userID, String passWord) {
        super();
        this.userID = userID;
        this.passWord = passWord;
    }

    public String getUserID() {
        return userID;
    }

    public void setUserID(String userID) {
        this.userID = userID;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }
}

```
```
loginForm.java
Tệp này cũng không cần thay đổi nhiều, nhưng đây là mã cho bạn tham khảo:
package fjs.cs.form;

import org.apache.struts.action.ActionForm;

public class loginForm extends ActionForm {
    private static final long serialVersionUID = 1L;
    private String userID; 
    private String passWord;

    public loginForm() {
        super();
    }

    public String getUserID() {
        return userID;
    }

    public void setUserID(String userID) {
        this.userID = userID;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }
}

```
```
struts-config.xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts-config PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN"
    "http://struts.apache.org/dtds/struts-config_1_3.dtd">

<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="fjs.cs.form.loginForm"/>
    </form-beans>

    <action-mappings>
        <action path="/login"
                type="fjs.cs.action.loginAction"
                name="loginForm"
                scope="request"
                input="/login.jsp">
            <forward name="success" path="/welcome.jsp"/>
            <forward name="failure" path="/login.jsp"/>
        </action>
    </action-mappings>
</struts-config>

```
```
web.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC 
    "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" 
    "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
    <display-name>HelloStruts1x</display-name>

    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Struts Action Servlet -->
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>

```
```
MstUser.hbm.xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="fjs.cs.model.MstUser" table="MSTUSERS">
        <id name="id" column="ID">
            <generator class="native"/>
        </id>
        <property name="userID" column="USERID"/>
        <property name="password" column="PASSWORD"/>
        <property name="deleteYmd" column="DELETE_YMD"/>
    </class>
</hibernate-mapping>

```
```
package fjs.cs.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            // Tạo một phiên làm việc từ cấu hình hibernate.cfg.xml
            return new Configuration().configure().buildSessionFactory();
        } catch (Throwable ex) {
            // Nếu không thể tạo phiên làm việc, ném ra ngoại lệ
            System.err.println("Initial SessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}

```
```
Struts_Spring_Hibernate/
├── src/
│   ├── fjs/
│   │   ├── cs/
│   │   │   ├── action/
│   │   │   │   ├── loginAction.java
│   │   │   ├── dao/
│   │   │   │   ├── loginDao.java
│   │   │   ├── dto/
│   │   │   │   ├── loginDto.java
│   │   │   ├── form/
│   │   │   │   ├── loginForm.java
│   │   │   ├── util/
│   │   │   │   ├── HibernateUtil.java
│   │   │   ├── db/
│   │   │   │   ├── connectDB.java (bỏ nếu không cần)
│   │   │   ├── hbm/
│   │   │   │   ├── MstUser.hbm.xml
│   ├── resources/
│   │   ├── hibernate.cfg.xml
│   │   ├── applicationContext.xml (nếu bạn dùng Spring)
├── WEB-INF/
│   ├── struts-config.xml
│   ├── web.xml
│   ├── login.jsp
│   ├── welcome.jsp

```






```
package fjs.cs.action;

import java.sql.SQLException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.form.loginForm;
import fjs.cs.handler.LoginHandler;

public class loginAction extends Action {

    private LoginHandler loginHandler = new LoginHandler(); // Khởi tạo handler ở đây

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest req, HttpServletResponse resp) throws SQLException {

        loginForm loginForm = (loginForm) form;
        String action = req.getParameter("action");

        if ("Login".equals(action)) {
            return loginHandler.handleLogin(mapping, req, loginForm);
        } 
        if ("Clear".equals(action)) {
            return loginHandler.handleClear(mapping, req, loginForm);
        }

        return mapping.findForward("failure");
    }
}

```
```
package fjs.cs.handler;

import java.sql.SQLException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.loginDao;
import fjs.cs.form.loginForm;

public class LoginHandler {
    
    private loginDao loginDao;

    public LoginHandler() {
        this.loginDao = new loginDao();
    }

    // Hàm xử lý đăng nhập
    public ActionForward handleLogin(ActionMapping mapping, HttpServletRequest req, loginForm loginForm) throws SQLException {
        String userid = loginForm.getUserID();
        String password = loginForm.getPassWord();
        ActionMessages errors = new ActionMessages();

        // Kiểm tra dữ liệu đầu vào
        if (isNullOrEmpty(userid)) {
            errors.add("username", new ActionMessage("error.username.required"));
        }
        if (isNullOrEmpty(password)) {
            errors.add("password", new ActionMessage("error.password.required"));
        }
        if (!errors.isEmpty()) {
            saveErrors(req, errors);
            return mapping.findForward("failure");
        }

        // Kiểm tra đăng nhập
        int cnt = loginDao.checkLogin(userid, password);
        if (cnt == 1) {
            HttpSession session = req.getSession();
            session.setAttribute("userid", userid);
            return mapping.findForward("success");
        } else {
            errors.add("login", new ActionMessage("error.login.invalid"));
            saveErrors(req, errors);
            return mapping.findForward("failure");
        }
    }

    // Hàm xử lý clear
    public ActionForward handleClear(ActionMapping mapping, HttpServletRequest req, loginForm loginForm) {
        req.setAttribute("errorMessageDiv", ""); // Reset thông báo lỗi
        loginForm.setUserID("");
        loginForm.setPassWord("");
        return mapping.findForward("failure");
    }

    // Hàm kiểm tra chuỗi rỗng hoặc null
    private boolean isNullOrEmpty(String str) {
        return str == null || str.trim().isEmpty();
    }
}

```
```
constants
package fjs.cs.util;

public class ActionConstants {
    public static final String ACTION_LOGIN = "Login";
    public static final String ACTION_CLEAR = "Clear";

    public static final String SESSION_USERID = "userid";

    public static final String ERROR_USERNAME_REQUIRED = "error.username.required";
    public static final String ERROR_PASSWORD_REQUIRED = "error.password.required";
    public static final String ERROR_LOGIN_INVALID = "error.login.invalid";

    public static final String FORWARD_SUCCESS = "success";
    public static final String FORWARD_FAILURE = "failure";
}

```
-----------------------------------------DAY 3 ----------------------------------------------------
```
xóa @id @entity
Thêm cấu hình cho Hibernate để sử dụng file ánh xạ XML:
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingResources">
        <list>
            <value>User.hbm.xml</value>
        </list>
    </property>
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
            <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
</bean>

```
```
Tạo file User.hbm.xml trong thư mục src/main/resources hoặc WEB-INF:
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="com.example.model.User" table="Users">
        <id name="username" column="username">
            <generator class="assigned"/>
        </id>
        <property name="password" column="password"/>
    </class>
</hibernate-mapping>
```
-----------------------------------------DAY 2 ----------------------------------------------------
```
web.xml
<web-app>
    <!-- Spring ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Struts ActionServlet -->
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```
```
struts-config.xml trong thư mục WEB-INF:
<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="com.example.form.LoginForm"/>
    </form-beans>

    <action-mappings>
        <action path="/login" type="com.example.action.LoginAction" 
                name="loginForm" scope="request" input="/jsp/login.jsp">
            <forward name="success" path="/jsp/welcome.jsp"/>
            <forward name="failure" path="/jsp/login.jsp"/>
        </action>
    </action-mappings>
</struts-config>
```
```
Tạo lớp LoginForm trong package com.example.form:
package com.example.form;

import org.apache.struts.action.ActionForm;

public class LoginForm extends ActionForm {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
```
Tạo lớp LoginAction trong package com.example.action:
package com.example.action;

import com.example.service.UserService;
import com.example.form.LoginForm;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.Action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class LoginAction extends Action {

    private UserService userService;

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        LoginForm loginForm = (LoginForm) form;

        String username = loginForm.getUsername();
        String password = loginForm.getPassword();

        if (userService.isValidUser(username, password)) {
            return mapping.findForward("success");
        } else {
            request.setAttribute("errorMessage", "Invalid username or password");
            return mapping.findForward("failure");
        }
    }
}
```
```
Tạo lớp UserService
Tạo lớp UserService trong package com.example.service:
package com.example.service;

import com.example.dao.UserDAO;
import com.example.model.User;

public class UserService {

    private UserDAO userDAO;

    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }

    public boolean isValidUser(String username, String password) {
        User user = userDAO.findByUsername(username);
        return user != null && user.getPassword().equals(password);
    }
}
```
```
Tạo lớp UserDAO trong package com.example.dao:
package com.example.dao;

import com.example.model.User;
import org.hibernate.SessionFactory;
import org.hibernate.Session;

public class UserDAO {

    private SessionFactory sessionFactory;

    public void setSessionFactory(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    public User findByUsername(String username) {
        Session session = sessionFactory.getCurrentSession();
        return session.createQuery("FROM User WHERE username = :username", User.class)
                      .setParameter("username", username)
                      .uniqueResult();
    }
}
```
```
Tạo lớp User trong package com.example.model:
package com.example.model;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class User {

    @Id
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
```
Tạo file applicationContext.xml trong thư mục WEB-INF:
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/your_database"/>
        <property name="username" value="your_username"/>
        <property name="password" value="your_password"/>
    </bean>

    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="com.example.model"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>

    <bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <!-- Cấu hình UserDAO và UserService -->
    <bean id="userDAO" class="com.example.dao.UserDAO">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <bean id="userService" class="com.example.service.UserService">
        <property name="userDAO" ref="userDAO"/>
    </bean>

    <!-- Cấu hình LoginAction -->
    <bean id="loginAction" class="com.example.action.LoginAction">
        <property name="userService" ref="userService"/>
    </bean>
</beans>
```
```
Tạo file login.jsp trong thư mục jsp:
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h2>Login Form</h2>
    <html:form action="/login">
        <html:text property="username" placeholder="Username"/>
        <html:text property="password" placeholder="Password" type="password"/>
        <html:submit value="Login"/>
    </html:form>
    <c:if test="${not empty errorMessage}">
        <p style="color:red;">${errorMessage}</p>
    </c:if>
</body>
</html>
```
```
Tạo file welcome.jsp trong thư mục jsp:
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h2>Welcome to the application!</h2>
</body>
</html>
```
```
Thay đổi phần cấu hình dataSource trong file applicationContext.xml để phù hợp với SQL Server:
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
    <property name="url" value="jdbc:sqlserver://localhost:1433;databaseName=your_database"/>
    <property name="username" value="your_username"/>
    <property name="password" value="your_password"/>
</bean>
```
```
Cũng cần thay đổi cấu hình cho Hibernate để sử dụng SQL Server:
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="packagesToScan" value="com.example.model"/>
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
            <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
</bean>
http://localhost:8080/your_project_name/jsp/login.jsp.
```
