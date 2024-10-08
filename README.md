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
