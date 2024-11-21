```
window.addEventListener("beforeunload", () => {
    fetch("/Hibernate_Spring/logoutTab.do?tabToken=" + currentTabToken, { method: "POST" });
});

```
```
dang nhap 1 lan 
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);
        String requestURI = httpRequest.getRequestURI();

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(requestURI)) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu session không tồn tại hoặc không có user
        if (session == null || session.getAttribute("userName") == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Resource not found");
            return;
        }

        // Lấy token từ URL
        String currentTabToken = httpRequest.getParameter("tabToken");
        if (currentTabToken == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Invalid tab token");
            return;
        }

        // Lấy danh sách token đã sử dụng từ session
        Set<String> usedTokens = (Set<String>) session.getAttribute("usedTokens");
        if (usedTokens == null) {
            usedTokens = new HashSet<>();
            session.setAttribute("usedTokens", usedTokens);
        }

        // Nếu token đã được sử dụng, trả về 404
        if (usedTokens.contains(currentTabToken)) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Token already in use");
            return;
        }

        // Đánh dấu token này là đang được sử dụng
        usedTokens.add(currentTabToken);

        // Cho phép tiếp tục xử lý request
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}

```
```
            // Đăng nhập thành công
        	String userName = loginLogic.saveUserName(userID, passWord);
        	session.setAttribute("userName", userName);
        	// Controller xử lý login thành công
        	String tabToken = (String) session.getAttribute("activeTabToken");
        	if (tabToken == null) {
        	    tabToken = UUID.randomUUID().toString();
        	    session.setAttribute("activeTabToken", tabToken);
        	}
        	try {
				response.sendRedirect("search.do?tabToken=" + tabToken);
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

            return null;
```
```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
@Autowired
private DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.jdbcAuthentication()
        .dataSource(dataSource)
        .usersByUsernameQuery("SELECT username, password, enabled FROM users WHERE username = ?")
        .authoritiesByUsernameQuery("SELECT username, authority FROM authorities WHERE username = ?");
}

```
```
Java Configuration (Thay thế spring-security.xml)
Bạn có thể cấu hình Spring Security bằng Java nếu không muốn dùng XML:
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/login.do", "/static/**").permitAll() // Không yêu cầu đăng nhập
                .antMatchers("/search.do").hasRole("USER")         // ROLE_USER mới được truy cập
                .antMatchers("/admin/**").hasRole("ADMIN")         // ROLE_ADMIN mới được truy cập
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login.do")                           // Trang đăng nhập
                .defaultSuccessUrl("/home.do")                   // Sau đăng nhập
                .failureUrl("/login.do?error=true")              // Lỗi đăng nhập
                .and()
            .logout()
                .logoutSuccessUrl("/login.do")                   // Sau khi logout
                .invalidateHttpSession(true);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("admin").password("{noop}password").roles("ADMIN")
            .and()
            .withUser("user").password("{noop}password").roles("USER");
    }
}

```
```
File spring-security.xml (hoặc cấu hình bằng Java)
Thêm file spring-security.xml trong WEB-INF:
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="
                http://www.springframework.org/schema/security
                http://www.springframework.org/schema/security/spring-security.xsd
                http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình xác thực đơn giản (In-Memory Authentication) -->
    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="admin" password="{noop}password" authorities="ROLE_ADMIN"/>
                <user name="user" password="{noop}password" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>

    <!-- Cấu hình URL bảo mật -->
    <http auto-config="true" use-expressions="true">
        <!-- Cho phép truy cập không cần đăng nhập -->
        <intercept-url pattern="/login.do" access="permitAll"/>
        <intercept-url pattern="/static/**" access="permitAll"/>

        <!-- Bảo vệ tài nguyên cần đăng nhập -->
        <intercept-url pattern="/search.do" access="hasRole('ROLE_USER')"/>
        <intercept-url pattern="/admin/**" access="hasRole('ROLE_ADMIN')"/>

        <!-- Form login -->
        <form-login login-page="/login.do" default-target-url="/home.do" authentication-failure-url="/login.do?error=true"/>
        
        <!-- Đăng xuất -->
        <logout logout-success-url="/login.do" invalidate-session="true"/>
    </http>
</beans:beans>

```
```
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-core</artifactId>
        <version>5.8.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.8.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
        <version>5.8.1</version>
    </dependency>
</dependencies>

```
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" id="WebApp_ID" version="4.0">
  	<display-name>CustomerSystem_Struts</display-name>
	<context-param>
	    <param-name>javax.servlet.jsp.jstl.fmt.encoding</param-name>
	    <param-value>UTF-8</param-value>
	</context-param>
	    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
    </context-param>
    <filter>
	    <filter-name>SessionFilter</filter-name>
	    <filter-class>fjs.cs.util.SessionFilter</filter-class>
	</filter>
	<filter-mapping>
	    <filter-name>SessionFilter</filter-name>
	    <url-pattern>*.do</url-pattern>
	</filter-mapping>
	    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

	<welcome-file-list>
	    <welcome-file>index.jsp</welcome-file> <!-- Đặt trang login.do làm welcome file -->
	</welcome-file-list>

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
</web-app>
```
```
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);
        String requestURI = httpRequest.getRequestURI();

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(requestURI)) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu session không tồn tại hoặc không có user
        if (session == null || session.getAttribute("userName") == null) {
            // Trả về mã lỗi 404
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Resource not found");
            return;
        }

        // Tiếp tục xử lý request nếu hợp lệ
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    // Kiểm tra xem URL có phải là tài nguyên công khai không
    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}



```
```
<script type="text/javascript">
    $(document).ready(function () {
        var selectedColumns = ${t}; // Danh sách cột đã chọn
        console.log("Selected Columns: ", selectedColumns);

        // Định nghĩa colModel
        const columnMapping = {
            "Customer ID": {
                name: "customerID",
                index: "customerID",
                width: 75,
                key: true,
                formatter: "showlink",
                formatoptions: {
                    baseLinkUrl: "edit.do",
                    idName: "id"
                }
            },
            "Customer Name": { name: "customerName", index: "customerName", width: 150 },
            "Sex": { name: "sex", index: "sex", width: 80, formatter: formatSex },
            "Birthday": { name: "birthday", index: "birthday", width: 100 },
            "Address": { name: "address", index: "address", width: 200 },
            "Email": { name: "email", index: "email", width: 200 }
        };

        var colNames = selectedColumns;
        var colModel = selectedColumns.map(column => columnMapping[column]);

        // Biến để lưu trạng thái sắp xếp nhiều cột
        var sortColumns = []; // Danh sách cột
        var sortOrders = [];  // Danh sách thứ tự sắp xếp

        // Tạo jqGrid với multisort
        $("#customerGrid").jqGrid({
            data: ${test},
            datatype: "local",
            colNames: colNames,
            colModel: colModel,
            pager: "#customerPager",
            rowNum: 10,
            multiSort: true, // Kích hoạt multisort
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true,
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons(); // Cập nhật trạng thái nút phân trang
            },
            onSortCol: function (index, order) {
                // Lưu trạng thái multisort
                const colIndex = sortColumns.indexOf(index);
                if (colIndex !== -1) {
                    // Nếu cột đã tồn tại, cập nhật thứ tự sắp xếp
                    sortOrders[colIndex] = order;
                } else {
                    // Nếu cột chưa tồn tại, thêm cột mới
                    sortColumns.push(index);
                    sortOrders.push(order);
                }
            }
        });

        // Cập nhật trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Chuyển trang giữ trạng thái multisort
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", {
                page: 1,
                sortname: sortColumns.join(","), // Nối danh sách cột
                sortorder: sortOrders.join(",") // Nối danh sách thứ tự sắp xếp
            }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage - 1,
                    sortname: sortColumns.join(","),
                    sortorder: sortOrders.join(",")
                }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage + 1,
                    sortname: sortColumns.join(","),
                    sortorder: sortOrders.join(",")
                }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", {
                page: lastPage,
                sortname: sortColumns.join(","),
                sortorder: sortOrders.join(",")
            }).trigger("reloadGrid");
        });

        // Formatter cho cột "Sex"
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }

        // Nút cài đặt header
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });

        console.log("${selectedColumns}");
    });
</script>

```
```
$(document).ready(function () {
    $("#customerGrid").jqGrid({
        url: 'search.do', // Endpoint server xử lý
        datatype: "json", // Dữ liệu trả về từ server dạng JSON
        mtype: "GET", // GET hoặc POST tùy thuộc vào server
        colNames: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
        colModel: [
            { name: "customerID", index: "customerID", width: 75, key: true },
            { name: "customerName", index: "customerName", width: 150 },
            { name: "sex", index: "sex", width: 80, formatter: formatSex },
            { name: "birthday", index: "birthday", width: 100 },
            { name: "address", index: "address", width: 200 },
            { name: "email", index: "email", width: 200 }
        ],
        pager: "#customerPager",
        rowNum: 10, // Số dòng trên mỗi trang
        rowList: [10, 20, 30], // Tùy chọn số dòng
        sortname: "customerID",
        sortorder: "asc",
        viewrecords: true,
        height: "auto",
        autowidth: true,
        multiselect: true, // Cho phép chọn nhiều dòng
        jsonReader: {
            root: "rows",
            page: "page",
            total: "total",
            records: "records",
            repeatitems: false,
            id: "customerID"
        },
        caption: "Customer List"
    });

    function formatSex(cellValue) {
        return cellValue === "0" ? "Male" : "Female";
    }

    $("#searchButton").click(function () {
        const searchParams = {
            customerName: $("#customerName").val(),
            sex: $("#sex").val(),
            birthdayFrom: $("#birthdayFrom").val(),
            birthdayTo: $("#birthdayTo").val()
        };

        $("#customerGrid").jqGrid("setGridParam", {
            url: 'search.do',
            datatype: "json",
            postData: searchParams, // Gửi tham số tìm kiếm
            page: 1 // Quay về trang đầu tiên
        }).trigger("reloadGrid");
    });
});

```
```
public class SearchAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        String customerName = request.getParameter("customerName");
        String sex = request.getParameter("sex");
        String fromBirthday = request.getParameter("birthdayFrom");
        String toBirthday = request.getParameter("birthdayTo");

        int page = Integer.parseInt(request.getParameter("page"));
        int rows = Integer.parseInt(request.getParameter("rows"));

        List<CustomerDto> customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);

        // Pagination logic
        int totalRecords = customerList.size();
        int totalPages = (int) Math.ceil((double) totalRecords / rows);

        int startRow = (page - 1) * rows;
        List<CustomerDto> paginatedList = customerList.subList(
            Math.min(startRow, totalRecords),
            Math.min(startRow + rows, totalRecords)
        );

        // Convert to jqGrid JSON format
        Gson gson = new Gson();
        String json = gson.toJson(new jqGridResult(totalRecords, totalPages, page, paginatedList));

        response.setContentType("application/json");
        response.getWriter().write(json);

        return null; // Vì đây là AJAX, không cần điều hướng
    }
}
	
```
```
public class jqGridResult {
    private int total; // Tổng số trang
    private int page; // Trang hiện tại
    private int records; // Tổng số bản ghi
    private List<?> rows; // Dữ liệu từng dòng

    public jqGridResult(int records, int total, int page, List<?> rows) {
        this.records = records;
        this.total = total;
        this.page = page;
        this.rows = rows;
    }

    // Getters và Setters
}

```
```
$("#customerGrid").jqGrid({
    // ... các cấu hình khác
    onSortCol: function (index, iCol, sortorder) {
        // Lưu trạng thái sort vào sessionStorage (hoặc localStorage)
        const sortState = {
            sortColumn: index,
            sortOrder: sortorder
        };
        sessionStorage.setItem("gridSortState", JSON.stringify(sortState));
    },
});

String sortColumn = request.getParameter("sidx");
String sortOrder = request.getParameter("sord");

// Sử dụng sortColumn và sortOrder trong truy vấn database
List<CustomerDto> sortedList = searchDao.searchCustomersSorted(customerName, sex, fromBirthday, toBirthday, sortColumn, sortOrder);
String lastSortColumn = (String) session.getAttribute("lastSortColumn");
String lastSortOrder = (String) session.getAttribute("lastSortOrder");

// Gửi giá trị này về JSP hoặc sử dụng trong logic backend.
String sortColumn = request.getParameter("sidx"); // Cột sắp xếp
String sortOrder = request.getParameter("sord"); // Hướng sắp xếp

// Lưu vào session
session.setAttribute("lastSortColumn", sortColumn);
session.setAttribute("lastSortOrder", sortOrder);
$(document).ready(function () {
    // Kiểm tra nếu đã lưu trạng thái sort
    const savedSortState = JSON.parse(sessionStorage.getItem("gridSortState"));
    let sortColumn = "customerID"; // Mặc định cột sắp xếp
    let sortOrder = "asc"; // Mặc định hướng sắp xếp

    if (savedSortState) {
        sortColumn = savedSortState.sortColumn;
        sortOrder = savedSortState.sortOrder;
    }

    // Khởi tạo jqGrid
    $("#customerGrid").jqGrid({
        // ... các cấu hình khác
        sortname: sortColumn, // Cột để sắp xếp
        sortorder: sortOrder, // Hướng sắp xếp
    });
});

```
```
$("#customerGrid").jqGrid({
    // Các cấu hình khác
    sortname: "customerID", // Giá trị mặc định
    sortorder: "asc",       // Giá trị mặc định
    onSortCol: function (index, sortorder) {
        sortColumn = index;
        sortOrder = sortorder;
    },
});
$(document).ready(function () {
    let sortColumn = "customerID"; // Cột mặc định để sắp xếp
    let sortOrder = "asc";        // Thứ tự mặc định

    // Khi người dùng thay đổi trạng thái sắp xếp
    $("#customerGrid").on("sortCol", function (e, index, sortorder) {
        sortColumn = index;
        sortOrder = sortorder;
    });

    // Cập nhật trạng thái sắp xếp khi load lại grid
    function reloadGridWithSort() {
        $("#customerGrid").jqGrid("setGridParam", {
            sortname: sortColumn,
            sortorder: sortOrder,
        }).trigger("reloadGrid");
    }

    // Cập nhật các nút phân trang
    function updatePaginationButtons() {
        const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
        const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
        $("#previousPageButton").prop("disabled", currentPage <= 1);
        $("#firstPageButton").prop("disabled", currentPage <= 1);
        $("#nextPageButton").prop("disabled", currentPage >= lastPage);
        $("#lastPageButton").prop("disabled", currentPage >= lastPage);
    }

    // Khi phân trang, giữ trạng thái sắp xếp
    $("#customerGrid").jqGrid("setGridParam", {
        onPaging: function () {
            setTimeout(function () {
                reloadGridWithSort();
                updatePaginationButtons();
            }, 100);
        }
    });

    // Thêm các nút điều hướng và gọi reloadGridWithSort
    $("#firstPageButton").click(function () {
        $("#customerGrid").jqGrid("setGridParam", { page: 1 });
        reloadGridWithSort();
    });

    $("#previousPageButton").click(function () {
        const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
        if (currentPage > 1) {
            $("#customerGrid").jqGrid("setGridParam", { page: currentPage - 1 });
            reloadGridWithSort();
        }
    });

    $("#nextPageButton").click(function () {
        const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
        const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
        if (currentPage < lastPage) {
            $("#customerGrid").jqGrid("setGridParam", { page: currentPage + 1 });
            reloadGridWithSort();
        }
    });

    $("#lastPageButton").click(function () {
        const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
        $("#customerGrid").jqGrid("setGridParam", { page: lastPage });
        reloadGridWithSort();
    });
});

```
```
<script type="text/javascript">
    $(document).ready(function () {
        var selectedColumns = ${t}; // Danh sách cột đã chọn
        console.log("Selected Columns: ", selectedColumns);

        // Định nghĩa colModel
        const columnMapping = {
            "Customer ID": {
                name: "customerID",
                index: "customerID",
                width: 75,
                key: true,
                formatter: "showlink",
                formatoptions: {
                    baseLinkUrl: "edit.do",
                    idName: "id"
                }
            },
            "Customer Name": { name: "customerName", index: "customerName", width: 150 },
            "Sex": { name: "sex", index: "sex", width: 80, formatter: formatSex },
            "Birthday": { name: "birthday", index: "birthday", width: 100 },
            "Address": { name: "address", index: "address", width: 200 },
            "Email": { name: "email", index: "email", width: 200 }
        };

        var colNames = selectedColumns;
        var colModel = selectedColumns.map(column => columnMapping[column]);

        // Biến để lưu trạng thái sắp xếp
        var sortColumn = "customerID"; // Giá trị mặc định
        var sortOrder = "asc";         // Giá trị mặc định

        // Tạo jqGrid
        $("#customerGrid").jqGrid({
            data: ${test},
            datatype: "local",
            colNames: colNames,
            colModel: colModel,
            pager: "#customerPager",
            rowNum: 10,
            sortname: sortColumn,
            sortorder: sortOrder,
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true,
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons(); // Cập nhật trạng thái nút phân trang
            },
            onSortCol: function (index, order) {
                sortColumn = index; // Lưu cột đang được sắp xếp
                sortOrder = order;  // Lưu thứ tự sắp xếp
            }
        });

        // Cập nhật trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Chuyển trang và giữ trạng thái sắp xếp
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", {
                page: 1,
                sortname: sortColumn,
                sortorder: sortOrder
            }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage - 1,
                    sortname: sortColumn,
                    sortorder: sortOrder
                }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage + 1,
                    sortname: sortColumn,
                    sortorder: sortOrder
                }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", {
                page: lastPage,
                sortname: sortColumn,
                sortorder: sortOrder
            }).trigger("reloadGrid");
        });

        // Xử lý formatter cho cột "Sex"
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }

        // Nút cài đặt header
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });

        console.log("${selectedColumns}");
    });
</script>

```
https://drive.google.com/drive/folders/1bPh22WOHnfjV8E2gzLIvzQr1Bx1uL9nx?usp=sharing
```
editaction
package fjs.cs.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionErrors;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;

public class EditAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
    	String customerId = request.getParameter("id"); 
    	System.out.println(customerId);
    	HttpSession session = request.getSession();
    	if (customerId != null && customerId != "") {
    		session.setAttribute("screen", "Edit");
    	}
    	else if (customerId == null) {
    		session.setAttribute("screen", "Add new");
    	}
    	String screen = (String) session.getAttribute("screen");
    	System.out.println(screen);
    	if ("Edit".equals(screen)) {
        	CustomerDto customer = searchDao.getCustomerById(Integer.parseInt(customerId));
        	request.setAttribute("customer", customer);
        	String birthday = request.getParameter("birthday");
        	String action = request.getParameter("action");
        	ActionErrors errors = new ActionErrors();
        	if("Save".equals(action)) {
        	    // Kiểm tra định dạng birthday
        	    if (birthday != null && !birthday.matches("^\\d{4}/\\d{2}/\\d{2}$")) {
        	        errors.add("birthday", new ActionMessage("error.birthday.invalid"));
        	    }

        	    // Nếu có lỗi, lưu lỗi và chuyển hướng lại form
        	    if (!errors.isEmpty()) {
        	        saveErrors(request, errors);
        	        return mapping.findForward("edit");
        	    }
        	}
    	} 	
    	return mapping.findForward("edit");
    }
}

```
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Edit Customer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
            margin: 0;
            padding: 20px;
        }

        .container {
            max-width: 600px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        h1 {
            text-align: center;
        }

        .form-group {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .form-group label {
            flex: 1;
            margin-right: 10px;
            font-weight: bold;
        }

        .form-group input[type="text"], 
        .form-group input[type="email"], 
        .form-group select, 
        .form-group textarea {
            flex: 2;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        textarea {
            resize: none;
        }

        .button-group {
            display: flex;
            justify-content: space-between;
            gap: 10px;
        }

        button {
            flex: 1;
            padding: 10px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }

        button:hover {
            background-color: #45a049;
        }

        .clear-button {
            background-color: #f44336;
        }

        .clear-button:hover {
            background-color: #e53935;
        }

        .error-message {
            color: red;
            text-align: center;
        }

        /* Breadcrumb and user welcome styling */
        .breadcrumb {
            padding: 10px 0;
            font-size: 14px;
        }

        .breadcrumb a {
            color: blue;
            text-decoration: none;
        }

        .breadcrumb a:hover {
            text-decoration: underline;
        }

        .welcome-user-container {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .divider {
            height: 2px;
            width: 100%;
            background-color: #ccc;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Breadcrumb and User Welcome Section -->
        <div class="breadcrumb"><a href="/Hibernate_Spring/search.do">Home</a> > Edit Customer</div>
		<div class="divider">${screen}</div>
        <div class="welcome-user-container">
            <p>Welcome, ${user}</p>
            <form action="search" method="post" style="display: inline;">
                <input type="hidden" name="action" value="logout" />
                <button type="submit">Log Out</button>
            </form>
        </div>



        <html:form action="/edit" method="post">
            <h1>Edit Customer</h1>
            <c:if test="${not empty errorMessage}">
                <p class="error-message">${errorMessage}</p>
            </c:if>

            <div class="form-group">
                <label for="id">Customer ID:</label>
                <input type="text" id="id" name="id" value="${customer.customerID}" readonly />
            </div>

            <div class="form-group">
                <label for="name">Customer Name:</label>
                <input type="text" id="name" name="name" value="${customer.customerName}" />
            </div>

            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" value="${customer.email}" />
            </div>

            <div class="form-group">
                <label for="sex">Sex:</label>
                <select id="sex" name="sex">
                    <option value="M" ${customer.sex == 'M' ? 'selected' : ''}>Male</option>
                    <option value="F" ${customer.sex == 'F' ? 'selected' : ''}>Female</option>
                </select>
            </div>

            <div class="form-group">
                <label for="birthday">Birthday:</label>
                <input type="text" id="birthday" name="birthday" value="${customer.birthday}" />
            </div>

            <div class="form-group">
                <label for="address">Address:</label>
                <textarea id="address" name="address" rows="3">${customer.address}</textarea>
            </div>

            <!-- Group of buttons: Save and Clear -->
            <div class="button-group">
                <button type="submit" name="action" value="Save">Save</button>
                <button type="reset" class="clear-button">Clear</button>
            </div>
            <html:errors/>
        </html:form>
    </div>
</body>
</html>

```
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<html>
<head>
    <title>Setting Header</title>
    <style>
        .selected {
            background-color: lightblue; /* Màu nền khi được chọn */
        }
        label {
            display: block; /* Hiển thị mỗi label trên 1 dòng */
            padding: 5px;
            cursor: pointer;
        }
        .list-container {
            display: flex;
            justify-content: space-between;
        }
        .list {
            width: 200px;
            height: 200px;
            border: 1px solid black;
            padding: 10px;
            overflow-y: auto;
        }
    </style>
    <script>
	    function checkButtonState() {
	        var customerList = document.getElementById('customerList');
	
	            // Nếu label được chọn là đầu tiên, disable Move Up
	            document.getElementById('moveUpBtn').disabled = (selectedLabel.previousElementSibling === null);
	
	            // Nếu label được chọn là cuối cùng, disable Move Down
	            document.getElementById('moveDownBtn').disabled = (selectedLabel.nextElementSibling === null);
	
	            // Các nút Move Left và Move Right chỉ cần kiểm tra nếu có label được chọn
	            document.getElementById('moveLeftBtn').disabled = false;
	            document.getElementById('moveRightBtn').disabled = false;
	    }
        function toggleMoveButtons() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            var moveUpButton = document.getElementById('moveUpButton');
            var moveDownButton = document.getElementById('moveDownButton');

            if (!selectedLabel) {
                return;
            }

            // Disable 'Move Up' if at the top
            if (!selectedLabel.previousElementSibling) {
                moveUpButton.disabled = true;
            } else {
                moveUpButton.disabled = false;
            }

            // Disable 'Move Down' if at the bottom
            if (!selectedLabel.nextElementSibling) {
                moveDownButton.disabled = true;
            } else {
                moveDownButton.disabled = false;
            }
        }
	    function selectLabel(label) {
	        // Bỏ chọn tất cả các label trong cả hai danh sách
	        var allLabels = document.querySelectorAll('.list label');
	        allLabels.forEach(function(item) {
	            item.classList.remove('selected');
	        });
	        // Chọn label hiện tại
	        label.classList.add('selected');
	    }

        function moveUp() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.previousElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel, selectedLabel.previousElementSibling);
                toggleMoveButtons();
            }
        }

        function moveDown() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.nextElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel.nextElementSibling, selectedLabel);
            }
        }

        function moveRight() {
            var selectedLabel = document.querySelector('#columnList label.selected');
            if (selectedLabel) {
                document.getElementById('customerList').appendChild(selectedLabel);
            }
        }

        function moveLeft() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel) {
                document.getElementById('columnList').appendChild(selectedLabel);
            } else {
                alert('Please select a label to move left.');
            }
        }
        function saveSelectedColumns() {
            var selectedColumns = [];
            var customerList = document.getElementById('customerList');
            var labels = customerList.querySelectorAll('label');
            
            labels.forEach(function(label) {
                selectedColumns.push(label.textContent.trim());
            });
            
            document.getElementById('selectedColumns').value = selectedColumns.join(',');
        }
    </script>
</head>
<body>
    <h2>Manage Search Screen Columns</h2>

    <html:form action="/upload">
        <div class="list-container">
            <!-- Column List -->
            <div id="columnList" class="list">
                <logic:iterate id="column" name="availableColumns">
                    <label onclick="selectLabel(this)">
                        <bean:write name="column" />
                    </label>
                </logic:iterate>
            </div>

            <!-- Move Buttons -->
            <div>
                <button type="button" onclick="moveUp()" >&#8594;</button><br><br>
                <button type="button" onclick="moveDown()">Move Down</button><br><br>
                <button type="button" onclick="moveRight()">Move Right</button><br><br>
                <button type="button" onclick="moveLeft()">Move Left</button>
            </div>

            <!-- Customer List -->
            <div id="customerList" class="list">
                <c:if test="${not empty sessionScope.selectedColumns}">
                    <c:forEach var="selectedColumn" items="${sessionScope.selectedColumns}">
                        <label onclick="selectLabel(this)">
                            <bean:write name="selectedColumn" />
                        </label>
                    </c:forEach>
                </c:if>
            </div>
        </div>

        <!-- Hidden input to store the selected columns -->
        <input type="hidden" name="selectedColumns" id="selectedColumns" 
               value="<%= request.getSession().getAttribute("selectedColumns") != null ? 
                       String.join(",", (String[]) request.getSession().getAttribute("selectedColumns")) : "conchonay" %>" />

        <br><br>
        <button type="submit" onclick="saveSelectedColumns()" name=action value="save">save</button>
    </html:form>
</body>
</html>

```
```
settingaction
package fjs.cs.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.dao.SearchDao;

public class UploadAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        // Danh sách tất cả các cột
    	HttpSession session = request.getSession();
        String[] allColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"};
        String selectedColumns = request.getParameter("selectedColumns");
        String action = request.getParameter("action");
        String[] avaiDefault = (String[]) session.getAttribute("availableColumns");
        if(avaiDefault==null) {
            List<String> defaultColumns = Arrays.asList("Customer ID", "Customer Name", "Sex", "Birthday", "Address");

            // Các cột còn lại (bao gồm "Email") sẽ là cột chưa được chọn
            List<String> availableList = new ArrayList<>();
            for (String column : allColumns) {
                if (!defaultColumns.contains(column)) {
                    availableList.add(column);
                }
            }

            // Lưu danh sách cột chưa được chọn vào session
            session.setAttribute("availableColumns", availableList.toArray(new String[0]));
        }
        List<String> selectedList = new ArrayList<>();
        List<String> availableList = new ArrayList<>();
        if("save".equals(action)) {
            // Chuyển danh sách cột đã chọn thành danh sách
            if (selectedColumns != null && !selectedColumns.isEmpty()) {
                selectedList = Arrays.asList(selectedColumns.split(","));
                // Lưu danh sách cột đã chọn vào session
                request.getSession().setAttribute("selectedColumns", selectedList.toArray(new String[0]));               
            }

            // Các cột chưa được chọn (cột ẩn) = Tất cả các cột - Cột đã chọn
            for (String column : allColumns) {
                if (!selectedList.contains(column)) {
                    availableList.add(column);
                    System.out.println(column);
                }
            }

            // Lưu danh sách cột ẩn vào session
            request.getSession().setAttribute("availableColumns", availableList.toArray(new String[0]));
            // Chuyển đến trang JSP setting header
            return mapping.findForward("search");
        }
//        else {
//            // Nếu action không phải là "save", thiết lập availableColumns là mảng rỗng
//            session.setAttribute("availableColumns", new String[0]);
//        }
        return mapping.findForward("success");
    }
}

```
```
searchjsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
		table {
		    width: 100%;
		    border-collapse: collapse;
		    table-layout :fixed;
		    border :  1px solid green;
		}
		#gbox_customerGrid {
			border :  2px solid green;
			border-collapse: collapse;
		}
		th,
		td {
		    padding: 8px;
		    text-align: left;
		    border-bottom: 1px solid #ddd;
		}
		th {
		    background-color: #f2f2f2;
		}
		tr:first-child > th {
		    background-color: rgb(0, 223, 0);
		    
		}
		tr:nth-child(even) {
		    background-color: #f2f2f2;
		}
    </style>
    
    <!-- Libraries for jqGrid and jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
        <html:errors/>
    </div>
    <div class="container">
    <html:form action="/search" method="post">
		<div class="search">
    <input type="text" id="customerName" name="customerName" placeholder="Customer Name"> <!-- Thêm name -->
    <select id="sex" name="sex"> <!-- Thêm name -->
        <option value=""></option>
        <option value="0">Male</option>
        <option value="1">Female</option>
    </select>
    <input type="text" id="birthdayFrom" name="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)"> <!-- Thêm name -->
    <input type="text" id="birthdayTo" name="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)"> <!-- Thêm name -->
		    <button id="searchButton" name="action" value="search" type="submit" >Search</button>
		    <button id="deleteButton" name="action" value="delete" type="submit">Delete Selected</button>
		    <button id="firstPageButton">First &lt;&lt;</button>
		    <button id="previousPageButton">Previous &lt;</button>
		    <button id="nextPageButton">Next &gt;</button>
			<button id="edit" name="action" value="edit" type="submit">edit</button>
		    <button id="">Last &gt;&gt;</button>
		    <button id="settingHeaderButton" name="action" value="upload" type="submit">Setting Header</button>
		    <input type="hidden" id="customerIds" name="customerIds" value="">
		</div>
	</html:form>

<table id="customerGrid"></table>
<div id="customerPager" style="display:none"></div>
<script type="text/javascript">
    $(document).ready(function () {
        var selectedColumns = ${t}; // Ví dụ: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"]
        console.log("Selected Columns: ", selectedColumns);

        // Tạo colNames và colModel động dựa trên selectedColumns
	    // Định nghĩa mapping object cho colModel
	    const columnMapping = {
        		
	    	    "Customer ID": {
	    	        name: "customerID",
	    	        index: "customerID",
	    	        width: 75,
	    	        key: true,
	    	        formatter: "showlink", // Sử dụng formatter "showlink"
	    	        formatoptions: {
	    	            baseLinkUrl: "edit.do", // Đường dẫn cơ bản
	    	            idName: "id" // Tên tham số query string
	    	        }
	    	    },
	        "Customer Name": { name: "customerName", index: "customerName", width: 150 },
	        "Sex": { name: "sex", index: "sex", width: 80, formatter: formatSex },
	        "Birthday": { name: "birthday", index: "birthday", width: 100 },
	        "Address": { name: "address", index: "address", width: 200 },
	        "Email": { name: "email", index: "email", width: 200 }
	    };
	
	    // Tạo colNames và colModel động dựa trên selectedColumns
	    var colNames = selectedColumns;
	    var colModel = selectedColumns.map(column => columnMapping[column]);

        // Cấu hình jqGrid
        $("#customerGrid").jqGrid({
            data: ${test}, // Dữ liệu
            datatype: "local",
            mtype: "POST",
            colNames: colNames, // Header cột động
            colModel: colModel, // Mô hình cột động
            pager: "#customerPager",
            rowNum: 10,
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true, // Cho phép chọn nhiều hàng
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons(); // Gọi hàm để cập nhật nút phân trang
            }
        });
        // Kiểm tra trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            // Disable nút Previous nếu ở trang đầu tiên
            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            // Disable nút Next nếu ở trang cuối cùng
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Gọi hàm updatePaginationButtons sau mỗi lần phân trang
        $("#customerGrid").jqGrid("setGridParam", {
            onPaging: function () {
                setTimeout(updatePaginationButtons, 100); // Delay ngắn để cập nhật sau khi phân trang
            }
        });


        // Delete button functionality
	    $("#deleteButton").click(function (event) {
	        const selectedIds = $("#customerGrid").jqGrid('getGridParam', 'selarrrow');
	        if (selectedIds.length === 0) {
	            event.preventDefault(); // Ngăn gửi form nếu không có gì được chọn
	            alert("Please select at least one customer to delete.");
	            return;
	        }
	        // Gán danh sách ID đã chọn vào input ẩn
	        $("#customerIds").val(selectedIds.join(","));
	    });

        // Pagination controls
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", { page: 1 }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage - 1 }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage + 1 }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", { page: lastPage }).trigger("reloadGrid");
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });
        console.log("${selectedColumns}")
    });
</script>


    </div>
</body>
</html>

```
```
searchaction
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionErrors;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import com.google.gson.Gson;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
		HttpSession session = request.getSession();
		System.out.println("aaaaaa");
		String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address"};
		if (session.getAttribute("selectedColumns") == null) {
			session.setAttribute("selectedColumns", defaultColumns);
		}
		Gson gsons = new Gson();
		String jsons = gsons.toJson(session.getAttribute("selectedColumns"));
		request.setAttribute("t", jsons);

		SearchForm searchForm = (SearchForm) form;
		String customerName = searchForm.getCustomerName();
		String sex = searchForm.getSex();
		String fromBirthday = searchForm.getBirthdayFrom();
		String toBirthday = searchForm.getBirthdayTo();
		ActionMessages errors = new ActionMessages();
		String action = request.getParameter("action");
		
		String sessionCustomerName = (String) session.getAttribute("lastValidCustomerName");
		String sessionSex = (String) session.getAttribute("lastValidSex");
		String sessionFromBirthday = (String) session.getAttribute("lastValidFromBirthday");
		String sessionToBirthday = (String) session.getAttribute("lastValidToBirthday");
		List<CustomerDto> customerList;
		customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
		Gson gson = new Gson();
		String json = gson.toJson(customerList);
		request.setAttribute("test", json);
		if("upload".equals(action)) {
			return mapping.findForward("settingheader");
		}
		if("edit".equals(action)) {
			return mapping.findForward("edit");
		}
		if ("delete".equals(action)) {
		    String deleteIDs = request.getParameter("customerIds");
		    if (deleteIDs != null && !deleteIDs.isEmpty()) {
		        // Tách các ID bằng dấu phẩy
		        String[] idArray = deleteIDs.split(",");
		        System.out.println(idArray);
		    }
		}
		System.out.println("quay lai 1");
		if("search".equals(action)) {
			System.out.println("search");
		    // Lưu giá trị search hiện tại vào session
		    session.setAttribute("lastValidCustomerName", customerName);
		    session.setAttribute("lastValidSex", sex); 
		    session.setAttribute("lastValidFromBirthday", fromBirthday);
		    session.setAttribute("lastValidToBirthday", toBirthday);

		    // Validate birthday format yyyy/mm/dd
		    if (fromBirthday != null && !fromBirthday.isEmpty() && !fromBirthday.matches("^\\d{4}/\\d{2}/\\d{2}$")) {
		        errors.add("birthdayFrom", new ActionMessage("error.birthday.invalid"));
		        saveErrors(request, errors);
		        
		        // Set lại giá trị vào form trước khi return
		        searchForm.setCustomerName(customerName);
		        searchForm.setSex(sex);
		        searchForm.setBirthdayFrom(fromBirthday); 
		        searchForm.setBirthdayTo(toBirthday);
		        return mapping.findForward("search");
		    }

		    if (toBirthday != null && !toBirthday.isEmpty() && !toBirthday.matches("^\\d{4}/\\d{2}/\\d{2}$")) {
		        errors.add("birthdayTo", new ActionMessage("error.birthday.invalid")); 
		        saveErrors(request, errors);
		        
		        // Set lại giá trị vào form trước khi return
		        searchForm.setCustomerName(customerName);
		        searchForm.setSex(sex);
		        searchForm.setBirthdayFrom(fromBirthday);
		        searchForm.setBirthdayTo(toBirthday);
		        
		        return mapping.findForward("search"); 
		    }

		    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
		    String jsons1 = gson.toJson(customerList);
		    request.setAttribute("test", jsons1);
		    return mapping.findForward("search");
		}
		System.out.println("quay lai");
		

		int PAGE_SIZE = 2;
		String pageStr = request.getParameter("page");
		Integer currentPage = 1;
		if (pageStr != null) {
		currentPage = Integer.parseInt(pageStr);
		}
		if (currentPage == null || currentPage < 1) {
		currentPage = 1;
		}
		
		int startRow = (currentPage - 1) * PAGE_SIZE;
		int totalCustomers = customerList.size();
		int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
		
		List<CustomerDto> paginatedList = customerList.subList(
		Math.min(startRow, totalCustomers),
		Math.min(startRow + PAGE_SIZE, totalCustomers)
		);

		
		session.setAttribute("customerList", paginatedList);
		session.setAttribute("currentPage", currentPage);
		session.setAttribute("totalPages", totalPages);
		
		// Lưu lại các giá trị tìm kiếm để hiển thị trên form
		searchForm.setCustomerName(sessionCustomerName);
		searchForm.setSex(sessionSex);
		searchForm.setBirthdayFrom(sessionFromBirthday);
		searchForm.setBirthdayTo(sessionToBirthday);
		
		return mapping.findForward("search");
		}

} 
```
------------ new day again --------------
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<html>
<head>
    <title>Setting Header</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqgrid/5.3.1/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqgrid/5.3.1/js/jquery.jqgrid.min.js"></script>
    <script>
        $(document).ready(function () {
            function loadColumns() {
                $.ajax({
                    url: '<c:url value="/upload.do?action=save" />',
                    method: 'GET',
                    dataType: 'json',
                    success: function (data) {
                        // Load các cột đã chọn và chưa chọn vào jqGrid
                        $("#selectedGrid").jqGrid('clearGridData').jqGrid('setGridParam', { data: data.selectedColumns }).trigger('reloadGrid');
                        $("#availableGrid").jqGrid('clearGridData').jqGrid('setGridParam', { data: data.availableColumns }).trigger('reloadGrid');
                    }
                });
            }

            // Khởi tạo jqGrid cho cột đã chọn
            $("#selectedGrid").jqGrid({
                datatype: 'local',
                colNames: ['Selected Columns'],
                colModel: [{ name: 'column', index: 'column', width: 200 }],
                height: 200,
                viewrecords: true,
                caption: "Selected Columns",
                onSelectRow: toggleMoveButtons
            });

            // Khởi tạo jqGrid cho cột chưa chọn
            $("#availableGrid").jqGrid({
                datatype: 'local',
                colNames: ['Available Columns'],
                colModel: [{ name: 'column', index: 'column', width: 200 }],
                height: 200,
                viewrecords: true,
                caption: "Available Columns",
                onSelectRow: toggleMoveButtons
            });

            // Nạp dữ liệu ban đầu
            loadColumns();

            // Các chức năng di chuyển cột
            $("#moveRightButton").click(function () {
                moveColumn("availableGrid", "selectedGrid");
            });

            $("#moveLeftButton").click(function () {
                moveColumn("selectedGrid", "availableGrid");
            });

            function moveColumn(fromGrid, toGrid) {
                var selRowId = $("#" + fromGrid).jqGrid('getGridParam', 'selrow');
                if (selRowId) {
                    var columnData = $("#" + fromGrid).jqGrid('getRowData', selRowId);
                    $("#" + toGrid).jqGrid('addRowData', selRowId, columnData);
                    $("#" + fromGrid).jqGrid('delRowData', selRowId);
                }
            }

            // Lưu cấu hình cột
            $("#saveButton").click(function () {
                var selectedColumns = $("#selectedGrid").jqGrid('getRowData').map(row => row.column).join(',');
                $.post('<c:url value="/upload.do?action=save" />', { selectedColumns: selectedColumns }, function () {
                    alert("Columns saved successfully!");
                });
            });
        });
    </script>
</head>
<body>
    <h2>Manage Search Screen Columns</h2>
    <html:form>
        <div style="display: flex; gap: 10px;">
            <table id="availableGrid"></table>
            <div style="display: flex; flex-direction: column; gap: 5px; justify-content: center;">
                <button type="button" id="moveRightButton">→</button>
                <button type="button" id="moveLeftButton">←</button>
            </div>
            <table id="selectedGrid"></table>
        </div>
        <br>
        <button type="button" id="saveButton">Save</button>
    </html:form>
</body>
</html>

```
```
import com.google.gson.Gson;
import com.google.gson.JsonObject;
import javax.servlet.http.HttpServletResponse;
// import các thư viện cần thiết

public class UploadAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        // Danh sách tất cả các cột
        String[] allColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"};
        String selectedColumns = request.getParameter("selectedColumns");
        String action = request.getParameter("action");

        // Khởi tạo JSON để gửi về
        Gson gson = new Gson();
        JsonObject jsonResponse = new JsonObject();

        if ("save".equals(action)) {
            // Xử lý lưu các cột đã chọn
            List<String> selectedList = selectedColumns != null ? Arrays.asList(selectedColumns.split(",")) : new ArrayList<>();
            List<String> availableList = new ArrayList<>();

            // Xác định các cột chưa chọn
            for (String column : allColumns) {
                if (!selectedList.contains(column)) {
                    availableList.add(column);
                }
            }

            // Đặt danh sách cột vào JSON để trả về cho jqGrid
            jsonResponse.add("selectedColumns", gson.toJsonTree(selectedList));
            jsonResponse.add("availableColumns", gson.toJsonTree(availableList));

            // Cấu hình response là JSON
            response.setContentType("application/json");
            response.getWriter().write(gson.toJson(jsonResponse));
            return null; // Không chuyển tiếp tới trang khác
        }

        // Trả về màn hình Setting Header nếu không phải lưu cột
        return mapping.findForward("settingHeader");
    }
}

```
```
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import com.google.gson.Gson;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
		HttpSession session = request.getSession();
		
		String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address","Email"};
		session.setAttribute("selectedColumns", defaultColumns);
		Gson gsons = new Gson();
		String jsons = gsons.toJson(session.getAttribute("selectedColumns"));
		request.setAttribute("t", jsons);
		System.out.println(jsons);
		SearchForm searchForm = (SearchForm) form;
		String customerName = searchForm.getCustomerName();
		String sex = searchForm.getSex();
		String fromBirthday = searchForm.getBirthdayFrom();
		String toBirthday = searchForm.getBirthdayTo();
		ActionMessages errors = new ActionMessages();
		String action = request.getParameter("action");
		
		String sessionCustomerName = (String) session.getAttribute("lastValidCustomerName");
		String sessionSex = (String) session.getAttribute("lastValidSex");
		String sessionFromBirthday = (String) session.getAttribute("lastValidFromBirthday");
		String sessionToBirthday = (String) session.getAttribute("lastValidToBirthday");
		
		List<CustomerDto> customerList;


		customerList = searchDao.searchCustomers(sessionCustomerName, sessionSex, sessionFromBirthday, sessionToBirthday);
		
		
		int PAGE_SIZE = 2;
		String pageStr = request.getParameter("page");
		Integer currentPage = 1;
		if (pageStr != null) {
		currentPage = Integer.parseInt(pageStr);
		}
		if (currentPage == null || currentPage < 1) {
		currentPage = 1;
		}
		
		int startRow = (currentPage - 1) * PAGE_SIZE;
		int totalCustomers = customerList.size();
		int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
		
		List<CustomerDto> paginatedList = customerList.subList(
		Math.min(startRow, totalCustomers),
		Math.min(startRow + PAGE_SIZE, totalCustomers)
		);
		Gson gson = new Gson();
		String json = gson.toJson(customerList);
		request.setAttribute("test", json);
		
		session.setAttribute("customerList", paginatedList);
		session.setAttribute("currentPage", currentPage);
		session.setAttribute("totalPages", totalPages);
		
		// Lưu lại các giá trị tìm kiếm để hiển thị trên form
		searchForm.setCustomerName(sessionCustomerName);
		searchForm.setSex(sessionSex);
		searchForm.setBirthdayFrom(sessionFromBirthday);
		searchForm.setBirthdayTo(sessionToBirthday);
		
		return mapping.findForward("search");
		}

}
```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
		table {
		    width: 100%;
		    border-collapse: collapse;
		    table-layout :fixed;
		    border :  1px solid green;
		}
		#gbox_customerGrid {
			border :  2px solid green;
			border-collapse: collapse;
		}
		th,
		td {
		    padding: 8px;
		    text-align: left;
		    border-bottom: 1px solid #ddd;
		}
		th {
		    background-color: #f2f2f2;
		}
		tr:first-child > th {
		    background-color: rgb(0, 223, 0);
		    
		}
		tr:nth-child(even) {
		    background-color: #f2f2f2;
		}
    </style>
    
    <!-- Libraries for jqGrid and jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
<div class="search">
    <input type="text" id="customerName" placeholder="Customer Name">
    <select id="sex">
        <option value="">Any</option>
        <option value="0">Male</option>
        <option value="1">Female</option>
    </select>
    <input type="text" id="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)">
    <input type="text" id="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)">
    <button id="searchButton">Search</button>
    <button id="deleteButton">Delete Selected</button>
    <button id="firstPageButton">First &lt;&lt;</button>
    <button id="previousPageButton">Previous &lt;</button>
    <button id="nextPageButton">Next &gt;</button>
    <button id="lastPageButton">Last &gt;&gt;</button>
    <button id="settingHeaderButton">Setting Header</button>
</div>
<div></div>
<table id="customerGrid"></table>
<div id="customerPager" style="display:none"></div>
<script type="text/javascript">
    $(document).ready(function () {
    	var selectedColumns = ${t};
    	console.log(selectedColumns);
        $("#customerGrid").jqGrid({
            data: ${test},
            datatype: "local",
            mtype: "POST",
            postData: function() {
                return {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                };
            },
            colNames: selectedColumns,
            colModel: [
                { name: "customerID", index: "customerID", width: 75, key: true },
                { name: "customerName", index: "customerName", width: 150 },
                { name: "sex", index: "sex", width: 80, formatter: formatSex },
                { name: "birthday", index: "birthday", width: 100 },
                { name: "address", index: "address", width: 200 },
                { name: "email", index: "email", width: 200 }
            ],
            pager: "#customerPager",
            rowNum: 10,
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true, // Enable multiple row selection
             // Enable alternate row colors // Apply the custom class for alternate rows
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons();
            }
        });
        // Kiểm tra trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            // Disable nút Previous nếu ở trang đầu tiên
            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            // Disable nút Next nếu ở trang cuối cùng
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Gọi hàm updatePaginationButtons sau mỗi lần phân trang
        $("#customerGrid").jqGrid("setGridParam", {
            onPaging: function () {
                setTimeout(updatePaginationButtons, 100); // Delay ngắn để cập nhật sau khi phân trang
            }
        });
        // Search button functionality
        $("#searchButton").click(function () {
            $("#customerGrid").jqGrid('setGridParam', {
                page: 1,
                postData: {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                }
            }).trigger("reloadGrid");
        });

        // Delete button functionality
        $("#deleteButton").click(function () {
            const selectedIds = $("#customerGrid").jqGrid('getGridParam', 'selarrrow');
            if (selectedIds.length === 0) {
                alert("Please select at least one customer to delete.");
                return;
            }

            if (confirm("Are you sure you want to delete the selected customers?")) {
                $.ajax({
                    url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/delete.do",
                    type: "POST",
                    data: { customerIds: selectedIds },
                    success: function(response) {
                        alert("Selected customers have been deleted successfully.");
                        $("#customerGrid").trigger("reloadGrid");
                    },
                    error: function() {
                        alert("An error occurred while deleting the customers.");
                    }
                });
            }
            
        });

        // Pagination controls
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", { page: 1 }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage - 1 }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage + 1 }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", { page: lastPage }).trigger("reloadGrid");
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });
        console.log("${selectedColumns}")
    });
</script>


    </div>
</body>
</html>

```
------ NEW DAY

```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
    </style>
    
    <!-- Libraries for jqGrid and jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="search">
		    <input type="text" id="customerName" placeholder="Customer Name">
		    <select id="sex">
		        <option value="">Any</option>
		        <option value="0">Male</option>
		        <option value="1">Female</option>
		    </select>
		    <input type="text" id="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)">
		    <input type="text" id="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)">
		    <button id="searchButton">Search</button>
		    <button id="deleteButton">Delete Selected</button>
		</div>
        <table id="customerGrid"></table>
        <div id="customerPager"></div>

<script type="text/javascript">
    $(document).ready(function () {
        $("#customerGrid").jqGrid({
            url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/search.do",
            datatype: "json",
            mtype: "POST",
            postData: function() {
                return {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                };
            },
            colNames: ["Select", "Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
            colModel: [
                { name: "select", index: "select", width: 50, align: "center", formatter: "checkbox", formatoptions: { disabled: false }},
                { name: "customerID", index: "customerID", width: 75, key: true },
                { name: "customerName", index: "customerName", width: 150 },
                { name: "sex", index: "sex", width: 80, formatter: formatSex },
                { name: "birthday", index: "birthday", width: 100 },
                { name: "address", index: "address", width: 200 },
                { name: "email", index: "email", width: 200 }
            ],
            pager: "#customerPager",
            rowNum: 10,
            rowList: [10, 20, 30],
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            caption: "Customer List",
            height: "auto",
            autowidth: true,
            multiselect: true, // Enable multiple row selection
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            }
        });

        // Search button functionality
        $("#searchButton").click(function () {
            $("#customerGrid").jqGrid('setGridParam', {
                page: 1,
                postData: {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                }
            }).trigger("reloadGrid");
        });

        // Delete button functionality
        $("#deleteButton").click(function () {
            const selectedIds = $("#customerGrid").jqGrid('getGridParam', 'selarrrow');
            if (selectedIds.length === 0) {
                alert("Please select at least one customer to delete.");
                return;
            }

            if (confirm("Are you sure you want to delete the selected customers?")) {
                $.ajax({
                    url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/delete.do",
                    type: "POST",
                    data: { customerIds: selectedIds },
                    success: function(response) {
                        alert("Selected customers have been deleted successfully.");
                        $("#customerGrid").trigger("reloadGrid");
                    },
                    error: function() {
                        alert("An error occurred while deleting the customers.");
                    }
                });
            }
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
    });
</script>

    </div>
</body>
</html>

```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
    </style>
    
    <!-- Thư viện jqGrid và jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <!-- jqGrid Table -->
        <div class="search">
		    <input type="text" id="customerName" placeholder="Customer Name">
		    <select id="sex">
		        <option value="">Any</option>
		        <option value="0">Male</option>
		        <option value="1">Female</option>
		    </select>
		    <input type="text" id="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)">
		    <input type="text" id="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)">
		    <button id="searchButton">Search</button>
		</div>
        <table id="customerGrid"></table>
        <div id="customerPager"></div>

<script type="text/javascript">
    $(document).ready(function () {
        $("#customerGrid").jqGrid({
            url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/search.do",
            datatype: "json",
            mtype: "POST",
            postData: function() {
                return {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                };
            },
            colNames: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
            colModel: [
                { name: "customerID", index: "customerID", width: 75, key: true },
                { name: "customerName", index: "customerName", width: 150 },
                { name: "sex", index: "sex", width: 80, formatter: formatSex },
                { name: "birthday", index: "birthday", width: 100 },
                { name: "address", index: "address", width: 200 },
                { name: "email", index: "email", width: 200 }
            ],
            pager: "#customerPager",
            rowNum: 10,
            rowList: [10, 20, 30],
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            caption: "Customer List",
            height: "auto",
            autowidth: true,
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            }
        });

        // Trigger search on input change or button click
        $("#searchButton").click(function () {
            $("#customerGrid").jqGrid('setGridParam', {
                page: 1,
                postData: {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                }
            }).trigger("reloadGrid");
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
    });
</script>

    </div>
</body>
</html>

```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Search Customers</title>
<style type="text/css">
    /* Add styles for jqGrid */
    .ui-jqgrid .ui-jqgrid-btable {
        table-layout: auto;
    }
    .ui-jqgrid-sortable {
        cursor: pointer;
    }
    .ui-jqgrid-pager {
        height: 25px;
    }
</style>

<!-- Include jQuery and jqGrid library -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jqGrid/4.6.0/js/jquery.jqGrid.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqGrid/4.6.0/css/ui.jqgrid.min.css">
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <html:errors/>
        <div class="divider"></div>
        <html:form action="/search" method="post">
        <div class="search">
            <!-- Search form controls -->
        </div>
        <div id="grid-container">
            <table id="customerGrid"></table>
            <div id="pager"></div>
        </div>
        <div style="padding-block: 20px; display: flex; gap: 20px">
            <button id="addNew" type="button">Add New</button>
            <button id="deleteButton" type="submit" name="action" value="delete">Delete</button>
        </div>
        </html:form>
    </div>

    <script>
        $(document).ready(function () {
            // Initialize jqGrid
            $("#customerGrid").jqGrid({
                url: "<c:url value='/search?action=data'/>",
                datatype: "json",
                colModel: [
                    { label: 'Customer ID', name: 'customerID', width: 100, key: true },
                    { label: 'Customer Name', name: 'customerName', width: 200 },
                    { label: 'Sex', name: 'sex', width: 100,
                        formatter: function (cellvalue, options, rowObject) {
                            return cellvalue === '0' ? 'Male' : 'Female';
                        }
                    },
                    { label: 'Birthday', name: 'birthday', width: 150 },
                    { label: 'Address', name: 'address', width: 300 },
                    { label: 'Email', name: 'email', width: 200 }
                ],
                pager: '#pager',
                rowNum: 10,
                rowList: [10, 20, 30],
                sortname: 'customerID',
                sortorder: "asc",
                viewrecords: true,
                gridview: true,
                caption: "Customer List"
            });
        });
    </script>
</body>
</html>
```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
    </style>
    
    <!-- Thư viện jqGrid và jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <html:errors/>
        <div class="divider"></div>

        <!-- Form Tìm kiếm -->
        <div class="search">
            <div>
                <label for="search">Customer Name:</label>
                <input type="text" id="customerName" name="customerName" />
            </div>
            <div>
                <label for="search">Sex:</label>
                <select id="sex" name="sex">
                    <option value=" "> </option>
                    <option value="0">Male</option>
                    <option value="1">Female</option>
                </select>
            </div>
            <div style="display: flex; justify-content: center; align-items: center;">
                <label for="search">Birthday:</label>
                <input type="text" id="birthdayFrom" name="birthdayFrom" />
                <p>~</p>
                <input type="text" id="birthdayTo" name="birthdayTo" />
            </div>
            <button type="button" onclick="reloadGrid()">Search</button>
            <button type="button" onclick="importCSV()">Import CSV</button>
            <button type="button" onclick="exportCSV()">Export CSV</button>
            <button type="button" onclick="openSettings()">SettingHeader</button>
        </div>

        <!-- jqGrid Table -->
        <table id="customerGrid"></table>
        <div id="customerPager"></div>

        <script type="text/javascript">
            $(document).ready(function () {
                // Cấu hình jqGrid
                $("#customerGrid").jqGrid({
                	url: window.location.pathname.substring(0, window.location.pathname.indexOf("/",2)) + "/search",
                    datatype: "json",
                    mtype: "POST",
                    postData: {
                        customerName: function () { return $("#customerName").val(); },
                        sex: function () { return $("#sex").val(); },
                        birthdayFrom: function () { return $("#birthdayFrom").val(); },
                        birthdayTo: function () { return $("#birthdayTo").val(); }
                    },
                    colNames: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
                    colModel: [
                        { name: "customerID", index: "customerID", width: 75, key: true },
                        { name: "customerName", index: "customerName", width: 150 },
                        { name: "sex", index: "sex", width: 80, formatter: formatSex },
                        { name: "birthday", index: "birthday", width: 100 },
                        { name: "address", index: "address", width: 200 },
                        { name: "email", index: "email", width: 200 }
                    ],
                    pager: "#customerPager",
                    rowNum: 10,
                    rowList: [10, 20, 30],
                    sortname: "customerID",
                    sortorder: "asc",
                    viewrecords: true,
                    caption: "Customer List",
                    height: "auto",
                    autowidth: true,
                    jsonReader: {
                        repeatitems: false,
                        root: "rows",
                        page: "page",
                        total: "total",
                        records: "records"
                    }
                });

                // Hàm định dạng cột Sex
                function formatSex(cellValue) {
                    return cellValue === "0" ? "Male" : "Female";
                }

                // Hàm tải lại lưới dữ liệu
                window.reloadGrid = function() {
                    $("#customerGrid").jqGrid("setGridParam", { page: 1 }).trigger("reloadGrid");
                };

                // Các hàm Import và Export CSV, mở màn hình Settings
                window.importCSV = function() {
                    // Viết logic import CSV tại đây
                    alert("Import CSV function called!");
                };

                window.exportCSV = function() {
                    // Viết logic export CSV tại đây
                    alert("Export CSV function called!");
                };

                window.openSettings = function() {
                    // Điều hướng đến trang cài đặt header
                    window.location.href = "${pageContext.request.contextPath}/settingHeader.jsp";
                };
            });
        </script>
    </div>
</body>
</html>

```
```
hibernate 4.
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;
import java.util.List;

public List<CustomerDto> getAllCustomers() {
    Session session = HibernateUtil.getSessionFactory().openSession();
    Transaction transaction = null;
    List<CustomerDto> customerList = null;

    try {
        transaction = session.beginTransaction();
        Query<CustomerDto> query = session.createQuery("FROM CustomerDto", CustomerDto.class);
        customerList = query.list();
        transaction.commit();
    } catch (Exception e) {
        if (transaction != null) {
            transaction.rollback();
        }
        e.printStackTrace();
    } finally {
        session.close();
    }
    
    return customerList;
}

```
```
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;
import java.util.List;

public List<CustomerDto> searchCustomers(String customerName, String sex, String fromBirthday, String toBirthday) {
    Session session = HibernateUtil.getSessionFactory().openSession();
    Transaction transaction = null;
    List<CustomerDto> customerList = null;

    try {
        transaction = session.beginTransaction();
        String hql = "FROM CustomerDto c WHERE 1=1"; // Bắt đầu với một điều kiện đúng

        if (customerName != null && !customerName.isEmpty()) {
            hql += " AND c.customerName LIKE :customerName";
        }
        if (sex != null && !sex.isEmpty()) {
            hql += " AND c.sex = :sex";
        }
        if (fromBirthday != null && !fromBirthday.isEmpty()) {
            hql += " AND c.birthday >= :fromBirthday";
        }
        if (toBirthday != null && !toBirthday.isEmpty()) {
            hql += " AND c.birthday <= :toBirthday";
        }

        Query<CustomerDto> query = session.createQuery(hql, CustomerDto.class);

        if (customerName != null && !customerName.isEmpty()) {
            query.setParameter("customerName", "%" + customerName + "%");
        }
        if (sex != null && !sex.isEmpty()) {
            query.setParameter("sex", sex);
        }
        if (fromBirthday != null && !fromBirthday.isEmpty()) {
            query.setParameter("fromBirthday", fromBirthday);
        }
        if (toBirthday != null && !toBirthday.isEmpty()) {
            query.setParameter("toBirthday", toBirthday);
        }

        customerList = query.list();
        transaction.commit();
    } catch (Exception e) {
        if (transaction != null) {
            transaction.rollback();
        }
        e.printStackTrace();
    } finally {
        session.close();
    }
    
    return customerList;
}

```
```
package fjs.cs.dao;

import java.util.ArrayList;
import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import fjs.cs.dto.CustomerDto;
import fjs.cs.model.Customer;

public class SearchDao {
	
	private SessionFactory sessionFactory;

	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}
    public List<CustomerDto> getAllCustomers() {
        Session session = sessionFactory.openSession();
        String hql = "SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address " +
                     "FROM Customer c WHERE c.delete_ymd IS NULL";
        
        List<?> results = session.createQuery(hql).list();
        session.close();

        List<CustomerDto> customers = new ArrayList<>();
        for (Object result : results) {
            Object[] row = (Object[]) result;
            CustomerDto dto = new CustomerDto();
            dto.setCustomerID((int) row[0]);
            dto.setCustomerName((String) row[1]);
            dto.setSex((String) row[2]);
            dto.setBirthday((String) row[3]);
            dto.setAddress((String) row[4]);
            customers.add(dto);
        }

        return customers;
    }
	
    public List<CustomerDto> searchCustomers(String customerName, String sex, String birthdayFrom, String birthdayTo) {
        Session session = sessionFactory.openSession();
        
        try {
            // Tạo câu truy vấn HQL
            StringBuilder hql = new StringBuilder("SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL");
            
            // Thêm điều kiện tìm kiếm
            if (customerName != null && !customerName.trim().isEmpty()) {
                hql.append(" AND c.customerName LIKE :customerName");
            }
            if (sex != null && !sex.trim().isEmpty()) {
                hql.append(" AND c.sex = :sex");
            }
            if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
                hql.append(" AND c.birthday >= :birthdayFrom");
            }
            if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
                hql.append(" AND c.birthday <= :birthdayTo");
            }

            // Tạo query
            Query query = session.createQuery(hql.toString());

            // Gán giá trị cho các tham số
            if (customerName != null && !customerName.trim().isEmpty()) {
                query.setParameter("customerName", "%" + customerName + "%");
            }
            if (sex != null && !sex.trim().isEmpty()) {
                query.setParameter("sex", sex);
            }
            if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
                query.setParameter("birthdayFrom", birthdayFrom);
            }
            if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
                query.setParameter("birthdayTo", birthdayTo);
            }

            // Lấy danh sách kết quả trả về
            List<?> results = query.list();
            List<CustomerDto> customerDtos = new ArrayList<>();

            // Xử lý kết quả trả về
            for (Object result : results) {
                Object[] row = (Object[]) result;
                CustomerDto dto = new CustomerDto(
                    (Integer) row[0],   // customerID
                    (String) row[1],    // customerName
                    (String) row[2],    // sex
                    (String) row[3],    // birthday
                    (String) row[4]     // address
                );
                customerDtos.add(dto);
            }
            
            return customerDtos;
        } finally {
            session.close();
        }
    }

	public void addCustomer(CustomerDto customerDto) {
	    Transaction transaction = null;
	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Bắt đầu transaction
	        transaction = session.beginTransaction();

	        // Tạo đối tượng Customer từ CustomerDto
	        Customer customer = new Customer();
	        customer.setCustomerID(customerDto.getCustomerID()); // Nếu ID tự động sinh, bạn có thể bỏ qua dòng này
	        customer.setCustomerName(customerDto.getCustomerName());
	        customer.setSex(customerDto.getSex());
	        customer.setBirthday(customerDto.getBirthday());
	        customer.setAddress(customerDto.getAddress());
	        customer.setEmail(customerDto.getEmail()); // Nếu có email trong Dto
	        
	        // Lưu đối tượng Customer vào database
	        session.save(customer);
	        
	        // Commit transaction
	        transaction.commit();
	        
	    } catch (Exception e) {
	        if (transaction != null) {
	            transaction.rollback(); // Rollback nếu có lỗi
	        }
	        e.printStackTrace();
	    } finally {
	        session.close(); // Đóng session sau khi hoàn thành
	    }
	}
    public boolean isCustomerExists(int customerId) {
    	Session session = sessionFactory.openSession();
        boolean exists = false;

        try {
            // HQL query để kiểm tra sự tồn tại của customerId
            String hql = "SELECT count(c.customerID) FROM Customer c WHERE c.customerID = :customerID AND c.delete_ymd IS NULL";
            Query query = session.createQuery(hql); // Không sử dụng kiểu tham số hóa ở đây
            query.setParameter("customerID", customerId);

            Long count = (Long) query.uniqueResult(); // Lấy kết quả duy nhất
            exists = (count != null && count > 0); // Nếu có kết quả và > 0, thì customerId tồn tại
        } catch (Exception e) {
            e.printStackTrace(); // Xử lý lỗi
        } finally {
            session.close(); // Đóng session
        }
        return exists;
    }
    public void editCustomer(CustomerDto customerDto) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();

        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Tìm đối tượng Customer từ database theo customerID
            Customer customer = (Customer) session.get(Customer.class, customerDto.getCustomerID());

            if (customer != null) {
                // Cập nhật thông tin từ CustomerDto
                customer.setCustomerName(customerDto.getCustomerName());
                customer.setSex(customerDto.getSex());
                customer.setBirthday(customerDto.getBirthday());
                customer.setAddress(customerDto.getAddress());
                customer.setEmail(customerDto.getEmail()); // Nếu có email trong Dto

                // Lưu đối tượng Customer đã được cập nhật vào database
                session.update(customer);

                // Commit transaction
                transaction.commit();
            } else {
                // Nếu không tìm thấy customer, có thể xử lý logic ở đây (ví dụ: throw exception)
                System.out.println("Customer with ID " + customerDto.getCustomerID() + " not found.");
            }

        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback(); // Rollback nếu có lỗi
            }
            e.printStackTrace();
        } finally {
            session.close(); // Đóng session sau khi hoàn thành
        }
    }
}

```
```
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
		HttpSession session = request.getSession();
		
		String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address"};
		if (session.getAttribute("selectedColumns") == null) {
		session.setAttribute("selectedColumns", defaultColumns);
		}
		
		SearchForm searchForm = (SearchForm) form;
		String customerName = searchForm.getCustomerName();
		String sex = searchForm.getSex();
		String fromBirthday = searchForm.getBirthdayFrom();
		String toBirthday = searchForm.getBirthdayTo();
		ActionMessages errors = new ActionMessages();
		String action = request.getParameter("action");
		
		String sessionCustomerName = (String) session.getAttribute("lastValidCustomerName");
		String sessionSex = (String) session.getAttribute("lastValidSex");
		String sessionFromBirthday = (String) session.getAttribute("lastValidFromBirthday");
		String sessionToBirthday = (String) session.getAttribute("lastValidToBirthday");
		
		List<CustomerDto> customerList;
		if("export".equals(action)) {
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Tạo file CSV
    	    StringBuilder csvData = new StringBuilder();
    	    
    	    // Tiêu đề của các cột
    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
    	    
    	    // Thêm dữ liệu khách hàng vào CSV
    	    for (CustomerDto customer : customerList) {
    	        csvData.append("\"").append(customer.getCustomerID()).append("\",")
    	               .append("\"").append(customer.getCustomerName()).append("\",")
    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
    	               .append("\"").append(customer.getBirthday()).append("\",")
    	               .append("\"").append(customer.getAddress()).append("\",")
    	               .append("\"").append(customer.getEmail()).append("\"\n");
    	    }
    	    
    	    // Đường dẫn đến nơi lưu file CSV
    	    String filePath = request.getServletContext().getRealPath("/") + "csv/Tests.csv";
    	    
    	    try {
    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
    	        System.out.println("File exported successfully to " + filePath);
    	    } catch (Exception e) {
    	        e.printStackTrace();
    	    }
		}
		if ("Search".equals(action)) {
		String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";
		if (!fromBirthday.isEmpty() && !fromBirthday.matches(datePattern)) {
		errors.add("birthday", new ActionMessage("error.birthday.invalid"));
		saveErrors(request, errors);
		
		searchForm.setBirthdayFrom(fromBirthday);
		customerList = searchDao.searchCustomers(sessionCustomerName, sessionSex, sessionFromBirthday, sessionToBirthday);
		} else {
		session.setAttribute("lastValidCustomerName", customerName);
		session.setAttribute("lastValidSex", sex);
		session.setAttribute("lastValidFromBirthday", fromBirthday);
		session.setAttribute("lastValidToBirthday", toBirthday);
		
		customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
		}
		} else {
		customerList = searchDao.searchCustomers(sessionCustomerName, sessionSex, sessionFromBirthday, sessionToBirthday);
		}
		
		int PAGE_SIZE = 2;
		String pageStr = request.getParameter("page");
		Integer currentPage = 1;
		if (pageStr != null) {
		currentPage = Integer.parseInt(pageStr);
		}
		if (currentPage == null || currentPage < 1) {
		currentPage = 1;
		}
		
		int startRow = (currentPage - 1) * PAGE_SIZE;
		int totalCustomers = customerList.size();
		int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
		
		List<CustomerDto> paginatedList = customerList.subList(
		Math.min(startRow, totalCustomers),
		Math.min(startRow + PAGE_SIZE, totalCustomers)
		);
		
		session.setAttribute("customerList", paginatedList);
		session.setAttribute("currentPage", currentPage);
		session.setAttribute("totalPages", totalPages);
		
		// Lưu lại các giá trị tìm kiếm để hiển thị trên form
		searchForm.setCustomerName(sessionCustomerName);
		searchForm.setSex(sessionSex);
		searchForm.setBirthdayFrom(sessionFromBirthday);
		searchForm.setBirthdayTo(sessionToBirthday);
		
		return mapping.findForward("search");
		}

}
```
```
if("export".equals(action)) {
    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);

    // Tạo file CSV
    StringBuilder csvData = new StringBuilder();

    // Tiêu đề của các cột
    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");

    // Thêm dữ liệu khách hàng vào CSV
    for (CustomerDto customer : customerList) {
        csvData.append("\"").append(customer.getCustomerID()).append("\",")
            .append("\"").append(customer.getCustomerName()).append("\",")
            .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
            .append("\"").append(customer.getBirthday()).append("\",")
            .append("\"").append(customer.getAddress()).append("\",")
            .append("\"").append(customer.getEmail()).append("\"\n");
    }

    // Lấy đường dẫn tương đối của thư mục CSV
    String csvDirPath = "csv";
    String fileName = "Test.csv";
    String filePath = csvDirPath + File.separator + fileName;

    try {
        // Tạo thư mục csv nếu chưa tồn tại
        new File(csvDirPath).mkdirs();

        // Ghi dữ liệu CSV vào file
        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
        System.out.println("File exported successfully to " + filePath);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Search Customers</title>
<style type="text/css">
    body {
        margin-left: 20px;
        margin-right: 20px;
        background-color: #bcffff;
    }
    .header {
        border-bottom: 2px solid;
    }
    .header h1 {
        color: red;
    }
    .welcome_user {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .divider {
        height: 20px;
        width: 100%;
        background-color: #3097ff;
    }
    .search form {
        display: flex;
        justify-content: space-around;
        align-items: center;
        padding: 10px;
        margin-top: 20px;
        background-color: #ffff56;
    }
    .pagination {
        margin-top: 10px;
    }
    .pagination .navigation-left {
        display: flex;
        gap: 5px;
        align-items: center;
    }
    .pagination .navigation-right {
        display: flex;
        gap: 5px;
        margin-left: auto;
        align-items: center;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        padding: 8px;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }
    th {
        background-color: #f2f2f2;
    }
    tr:first-child > th {
        background-color: rgb(0, 223, 0);
    }
    tr:nth-child(even) {
        background-color: #f2f2f2;
    }
</style>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <html:errors/>
        <div class="divider"></div>
        <html:form action="/search" method="post">
        <div class="search">
            <div>
                <label for="search">Customer Name:</label>
                <html:text property="customerName" />
            </div>
            <div>
                <label for="search">Sex:</label>
                <html:select property="sex">
                    <html:option value=" "></html:option>
                    <html:option value="0">Male</html:option>
                    <html:option value="1">Female</html:option>
                </html:select>
            </div>
            <div style="display: flex; justify-content: center; align-items: center;">
                <label for="search">Birthday:</label>
                <html:text property="birthdayFrom" styleClass="birthday" />
                <p>~</p>
                <html:text property="birthdayTo" styleClass="birthday" />
            </div>
            <button type="submit" value="Search" name="action">Search</button>
            <button type="submit" value="import" name="action">Import CSV</button>
            <button type="submit" value="SettingHeader" name="action">SettingHeader</button>
        	<button type="submit" value="export" name="action">Export CSV</button>
        </div>
        <div class="pagination">
            <div class="navigation-left">
                <logic:present name="currentPage">
                    <logic:greaterThan name="currentPage" value="1">
                        <button type="submit" name="page" value="1">&laquo;</button>
                    </logic:greaterThan>
                    <logic:lessEqual name="currentPage" value="1">
                        <button type="button" disabled>&laquo;</button>
                    </logic:lessEqual>
                </logic:present>

                <logic:present name="currentPage">
                    <logic:greaterThan name="currentPage" value="1">
                        <button type="submit" name="page" value="${currentPage - 1}">&lt;</button>
                    </logic:greaterThan>
                    <logic:lessEqual name="currentPage" value="1">
                        <button type="button" disabled>&lt;</button>
                    </logic:lessEqual>
                </logic:present>
                <p>Previous</p>
            </div>
            <div class="navigation-right">
                <p>Next</p>
                <logic:present name="currentPage">
                    <logic:lessThan name="currentPage" value="${totalPages}">
                        <button type="submit" name="page" value="${currentPage + 1}">&gt;</button>
                    </logic:lessThan>
                    <logic:greaterEqual name="currentPage" value="${totalPages}">
                        <button type="button" disabled>&gt;</button>
                    </logic:greaterEqual>
                </logic:present>

                <logic:present name="currentPage">
                    <logic:lessThan name="currentPage" value="${totalPages}">
                        <button type="submit" name="page" value="${totalPages}">&raquo;</button>
                    </logic:lessThan>
                    <logic:greaterEqual name="currentPage" value="${totalPages}">
                        <button type="button" disabled>&raquo;</button>
                    </logic:greaterEqual>
                </logic:present>
            </div>
        </div>
        <div class="table">
            <table>
                <tr>
                    <th><input type="checkbox" id="select-all" onclick="selectAllCheckboxes(this)"></th>
                    <logic:iterate id="column" name="selectedColumns">
                        <th><bean:write name="column" /></th>
                    </logic:iterate>
                </tr>
                <logic:iterate id="customer" name="customerList">
                    <tr>
                        <td><input type="checkbox" name="deleteIds" value="${customer.customerID}" onclick="updateSelectAll()"></td>
                        <logic:iterate id="column" name="selectedColumns" scope="session">
                            <td>
                                <logic:equal name="column" value="Customer ID">
                                    <a href="edit?id=${customer.customerID}"><bean:write name="customer" property="customerID" /></a>
                                </logic:equal>
                                <logic:equal name="column" value="Customer Name">
                                    <bean:write name="customer" property="customerName" />
                                </logic:equal>
                                <logic:equal name="column" value="Sex">
                                    <logic:equal name="customer" property="sex" value="0">Male</logic:equal>
                                    <logic:equal name="customer" property="sex" value="1">Female</logic:equal>
                                </logic:equal>
                                <logic:equal name="column" value="Birthday">
                                    <bean:write name="customer" property="birthday" />
                                </logic:equal>
                                <logic:equal name="column" value="Address">
                                    <bean:write name="customer" property="address" />
                                </logic:equal>
                                <logic:equal name="column" value="Email">
                                    <bean:write name="customer" property="email" />
                                </logic:equal>
                            </td>
                        </logic:iterate>
                    </tr>
                </logic:iterate>
            </table>
            <div style="padding-block: 20px; display: flex; gap: 20px">
                <button id="addNew" type="button">Add New</button>
                <button id="deleteButton" type="submit" name="action" value="delete">Delete</button>
            </div>
        </div>
        </html:form>
    </div>
</body>
</html>
```
https://drive.google.com/drive/folders/1bPh22WOHnfjV8E2gzLIvzQr1Bx1uL9nx?usp=drive_link
```
search action
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
	public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
    	HttpSession session = request.getSession();
    	
        String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address"};
        
        // Nếu selectedColumns chưa được lưu trong session thì thiết lập mặc định
        if (session.getAttribute("selectedColumns") == null) {
            session.setAttribute("selectedColumns", defaultColumns);
        } else {
            // Nếu đã có selectedColumns trong session, sử dụng danh sách hiện tại
            String[] selectedColumns = (String[]) session.getAttribute("selectedColumns");
            session.setAttribute("selectedColumns", selectedColumns);
            // Nếu selectedColumns rỗng hoặc không hợp lệ, thiết lập lại mặc định
            if (selectedColumns.length == 0) {
                session.setAttribute("selectedColumns", defaultColumns);
            }
        }
        
//        String[] selected = (String[]) session.getAttribute("selectedColumns");
//        System.out.println("con day " +selected);

    	SearchForm searchForm = (SearchForm) form;
    	String customerName = searchForm.getCustomerName();
        String sex = searchForm.getSex();
        String fromBirthday = searchForm.getBirthdayFrom();
        String toBirthday = searchForm.getBirthdayTo();
        ActionMessages errors = new ActionMessages();
        
        String action = request.getParameter("action");
    	List<CustomerDto> customerList;

    	if ("export".equals(action)) {
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Tạo file CSV
    	    StringBuilder csvData = new StringBuilder();
    	    
    	    // Tiêu đề của các cột
    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
    	    
    	    // Thêm dữ liệu khách hàng vào CSV
    	    for (CustomerDto customer : customerList) {
    	        csvData.append("\"").append(customer.getCustomerID()).append("\",")
    	               .append("\"").append(customer.getCustomerName()).append("\",")
    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
    	               .append("\"").append(customer.getBirthday()).append("\",")
    	               .append("\"").append(customer.getAddress()).append("\",")
    	               .append("\"").append(customer.getEmail()).append("\"\n");
    	    }
    	    
    	    // Đường dẫn đến nơi lưu file CSV
    	    String filePath = "C:/Users/ASUS/Desktop/Data/customer_export.csv";
    	    
    	    try {
    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
    	        System.out.println("File exported successfully to " + filePath);
    	    } catch (Exception e) {
    	        e.printStackTrace();
    	    }
    	    
    	    // Chuyển hướng hoặc thông báo thành công (có thể thêm redirect hoặc thông báo trong giao diện)
    	    return mapping.findForward("search");
    	}
    	
    	if("SettingHeader".equals(action)) {
    		System.out.println("day la "+action);
    		return mapping.findForward("settingheader");
    	}

    	if ("Search".equals(action)) {
    	    // Kiểm tra định dạng YYYY/MM/DD của birthdayFrom và birthdayTo
    	    String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";  // Regex kiểm tra định dạng YYYY/MM/DD}
    	    
    	    // Không có lỗi, thực hiện tìm kiếm
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Logic phân trang...
    	    int PAGE_SIZE = 2;
    	    String pageStr = request.getParameter("page");
    	    Integer currentPage = 1;
    	    if (pageStr != null) {
    	        currentPage = Integer.parseInt(pageStr);
    	    }
    	    if (currentPage == null || currentPage < 1) {
    	        currentPage = 1;
    	    }
    	    int startRow = (currentPage - 1) * PAGE_SIZE;
    	    int totalCustomers = customerList.size();
    	    int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
    	    
    	    List<CustomerDto> paginatedList = customerList.subList(
    	        Math.min(startRow, totalCustomers),
    	        Math.min(startRow + PAGE_SIZE, totalCustomers)
    	    );
    	    
    	    // Lưu thông tin tìm kiếm vào session
    	    session.setAttribute("customerName", customerName);
    	    session.setAttribute("sex", sex);
    	    session.setAttribute("fromBirthday", fromBirthday);
    	    session.setAttribute("toBirthday", toBirthday);
    	    
    	    session.setAttribute("customerList", paginatedList);
    	    session.setAttribute("currentPage", currentPage);
    	    session.setAttribute("totalPages", totalPages);
    	    
    	    return mapping.findForward("search");
    	}


        if (customerName != null || sex != null || fromBirthday != null || toBirthday != null) {
            customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
        } else {	
            customerList = searchDao.getAllCustomers();
        }
        int PAGE_SIZE = 2;
        String pageStr = request.getParameter("page");
        Integer currentPage = (Integer) session.getAttribute("currentPage");
        if (pageStr != null) {
            currentPage = Integer.parseInt(pageStr);
        }
        if (currentPage == null || currentPage < 1 ) {
            currentPage = 1;
        }
        int startRow = (currentPage - 1) * PAGE_SIZE;
        int totalCustomers = customerList.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
        List<CustomerDto> paginatedList = customerList.subList(
                Math.min(startRow, totalCustomers),
                Math.min(startRow + PAGE_SIZE, totalCustomers)
        );
        session.setAttribute("customerName", customerName);
        session.setAttribute("sex", sex);
        session.setAttribute("fromBirthday", fromBirthday);
        session.setAttribute("toBirthday", toBirthday);
        session.setAttribute("customerList", paginatedList);
        session.setAttribute("currentPage", currentPage);
        session.setAttribute("totalPages", totalPages);
    	return mapping.findForward("search");
    }
}

```
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Search Customers</title>
<style type="text/css">
    body {
        margin-left: 20px;
        margin-right: 20px;
        background-color: #bcffff;
    }
    .header {
        border-bottom: 2px solid;
    }
    .header h1 {
        color: red;
    }
    .welcome_user {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .divider {
        height: 20px;
        width: 100%;
        background-color: #3097ff;
    }
    .search form {
        display: flex;
        justify-content: space-around;
        align-items: center;
        padding: 10px;
        margin-top: 20px;
        background-color: #ffff56;
    }
    .pagination form {
        margin-top: 10px;
        display: flex;
    }
    .pagination .navigation-left {
        display: flex;
        gap: 5px;
        align-items: center;
    }
    .pagination .navigation-right {
        display: flex;
        gap: 5px;
        margin-left: auto;
        align-items: center;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        padding: 8px;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }
    th {
        background-color: #f2f2f2;
    }
    tr:first-child > th {
        background-color: rgb(0, 223, 0);
    }
    tr:nth-child(even) {
        background-color: #f2f2f2;
    }
</style>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <div class="divider"></div>
        <div class="search">
            <html:form action="/search" method="post">
                <div>
                    <label for="search">Customer Name:</label>
                    <html:text property="customerName" />
                </div>
                <div>
                    <label for="search">Sex:</label>
                    <html:select property="sex">
                        <html:option value=" "></html:option>
                        <html:option value="0">Male</html:option>
                        <html:option value="1">Female</html:option>
                    </html:select>
                </div>
                <div style="display: flex; justify-content: center; align-items: center;">
                    <label for="search">Birthday:</label>
                    <html:text property="birthdayFrom" styleClass="birthday" />
                    <p>~</p>
                    <html:text property="birthdayTo" styleClass="birthday" />
                </div>
                <button type="submit" value="Search" name="action">Search</button>
                <!-- Export Button -->
                <button type="submit" value="export" name="action">Export CSV</button>
                <button type="submit" value="SettingHeader" name="action">SettingHeader</button>
            </html:form>
        </div>
        <div class="pagination">
            <html:form action="/search" method="post">
                <html:hidden property="customerName" value="${customerName}" />
                <html:hidden property="sex" value="${sex}" />
                <html:hidden property="birthdayFrom" value="${fromBirthday}" />
                <html:hidden property="birthdayTo" value="${toBirthday}" />
                <div class="navigation-left">
                    <logic:present name="currentPage">
                        <logic:greaterThan name="currentPage" value="1">
                            <button type="submit" name="page" value="1">&laquo;</button>
                        </logic:greaterThan>
                        <logic:lessEqual name="currentPage" value="1">
                            <button type="button" disabled>&laquo;</button>
                        </logic:lessEqual>
                    </logic:present>

                    <logic:present name="currentPage">
                        <logic:greaterThan name="currentPage" value="1">
                            <button type="submit" name="page" value="${currentPage - 1}">&lt;</button>
                        </logic:greaterThan>
                        <logic:lessEqual name="currentPage" value="1">
                            <button type="button" disabled>&lt;</button>
                        </logic:lessEqual>
                    </logic:present>
                    <p>Previous</p>
                </div>
                <div class="navigation-right">
                    <p>Next</p>
                    <logic:present name="currentPage">
                        <logic:lessThan name="currentPage" value="${totalPages}">
                            <button type="submit" name="page" value="${currentPage + 1}">&gt;</button>
                        </logic:lessThan>
                        <logic:greaterEqual name="currentPage" value="${totalPages}">
                            <button type="button" disabled>&gt;</button>
                        </logic:greaterEqual>
                    </logic:present>

                    <logic:present name="currentPage">
                        <logic:lessThan name="currentPage" value="${totalPages}">
                            <button type="submit" name="page" value="${totalPages}">&raquo;</button>
                        </logic:lessThan>
                        <logic:greaterEqual name="currentPage" value="${totalPages}">
                            <button type="button" disabled>&raquo;</button>
                        </logic:greaterEqual>
                    </logic:present>
                </div>
            </html:form>
        </div>
        <div class="table">
            <html:form action="/search" method="post" onsubmit="return validateDelete();">
                <table>
				    <tr>
				        <th><input type="checkbox" id="select-all" onclick="selectAllCheckboxes(this)"></th>
				        <logic:iterate id="column" name="selectedColumns">
				            <th><bean:write name="column" /></th>
				        </logic:iterate>
				    </tr>
				    <logic:iterate id="customer" name="customerList">
				        <tr>
				            <td><input type="checkbox" name="deleteIds" value="${customer.customerID}" onclick="updateSelectAll()"></td>
				            <logic:iterate id="column" name="selectedColumns" scope="session">
				                <td>
				                    <logic:equal name="column" value="Customer ID">
				                        <a href="edit?id=${customer.customerID}"><bean:write name="customer" property="customerID" /></a>
				                    </logic:equal>
				                    <logic:equal name="column" value="Customer Name">
				                        <bean:write name="customer" property="customerName" />
				                    </logic:equal>
				                    <logic:equal name="column" value="Sex">
				                        <logic:equal name="customer" property="sex" value="0">Male</logic:equal>
				                        <logic:equal name="customer" property="sex" value="1">Female</logic:equal>
				                    </logic:equal>
				                    <logic:equal name="column" value="Birthday">
				                        <bean:write name="customer" property="birthday" />
				                    </logic:equal>
				                    <logic:equal name="column" value="Address">
				                        <bean:write name="customer" property="address" />
				                    </logic:equal>
				                    <logic:equal name="column" value="Email">
				                        <bean:write name="customer" property="email" />
				                    </logic:equal>
				                </td>
				            </logic:iterate>
				        </tr>
				    </logic:iterate>
                </table>
                <div style="padding-block: 20px; display: flex; gap: 20px">
                    <button id="addNew" type="button">Add New</button>
                    <button id="deleteButton" type="submit" name="action" value="delete">Delete</button>
                </div>
            </html:form>
        </div>
	
    </div>
</body>
</html>

```
```
upload.jsp
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<html>
<head>
    <title>Setting Header</title>
    <style>
        .selected {
            background-color: lightblue; /* Màu nền khi được chọn */
        }
        label {
            display: block; /* Hiển thị mỗi label trên 1 dòng */
            padding: 5px;
            cursor: pointer;
        }
        .list-container {
            display: flex;
            justify-content: space-between;
        }
        .list {
            width: 200px;
            height: 200px;
            border: 1px solid black;
            padding: 10px;
            overflow-y: auto;
        }
    </style>
    <script>
	    function checkButtonState() {
	        var customerList = document.getElementById('customerList');
	
	            // Nếu label được chọn là đầu tiên, disable Move Up
	            document.getElementById('moveUpBtn').disabled = (selectedLabel.previousElementSibling === null);
	
	            // Nếu label được chọn là cuối cùng, disable Move Down
	            document.getElementById('moveDownBtn').disabled = (selectedLabel.nextElementSibling === null);
	
	            // Các nút Move Left và Move Right chỉ cần kiểm tra nếu có label được chọn
	            document.getElementById('moveLeftBtn').disabled = false;
	            document.getElementById('moveRightBtn').disabled = false;
	    }
        function toggleMoveButtons() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            var moveUpButton = document.getElementById('moveUpButton');
            var moveDownButton = document.getElementById('moveDownButton');

            if (!selectedLabel) {
                return;
            }

            // Disable 'Move Up' if at the top
            if (!selectedLabel.previousElementSibling) {
                moveUpButton.disabled = true;
            } else {
                moveUpButton.disabled = false;
            }

            // Disable 'Move Down' if at the bottom
            if (!selectedLabel.nextElementSibling) {
                moveDownButton.disabled = true;
            } else {
                moveDownButton.disabled = false;
            }
        }
	    function selectLabel(label) {
	        // Bỏ chọn tất cả các label trong cả hai danh sách
	        var allLabels = document.querySelectorAll('.list label');
	        allLabels.forEach(function(item) {
	            item.classList.remove('selected');
	        });
	        // Chọn label hiện tại
	        label.classList.add('selected');
	    }

        function moveUp() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.previousElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel, selectedLabel.previousElementSibling);
                toggleMoveButtons();
            }
        }

        function moveDown() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.nextElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel.nextElementSibling, selectedLabel);
            }
        }

        function moveRight() {
            var selectedLabel = document.querySelector('#columnList label.selected');
            if (selectedLabel) {
                document.getElementById('customerList').appendChild(selectedLabel);
            }
        }

        function moveLeft() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel) {
                document.getElementById('columnList').appendChild(selectedLabel);
            } else {
                alert('Please select a label to move left.');
            }
        }
        function saveSelectedColumns() {
            var selectedColumns = [];
            var customerList = document.getElementById('customerList');
            var labels = customerList.querySelectorAll('label');
            
            labels.forEach(function(label) {
                selectedColumns.push(label.textContent.trim());
            });
            
            document.getElementById('selectedColumns').value = selectedColumns.join(',');
        }
    </script>
</head>
<body>
    <h2>Manage Search Screen Columns</h2>

    <html:form action="/upload">
        <div class="list-container">
            <!-- Column List -->
            <div id="columnList" class="list">
                <logic:iterate id="column" name="availableColumns">
                    <label onclick="selectLabel(this)">
                        <bean:write name="column" />
                    </label>
                </logic:iterate>
            </div>

            <!-- Move Buttons -->
            <div>
                <button type="button" onclick="moveUp()">Move Up</button><br><br>
                <button type="button" onclick="moveDown()">Move Down</button><br><br>
                <button type="button" onclick="moveRight()">Move Right</button><br><br>
                <button type="button" onclick="moveLeft()">Move Left</button>
            </div>

            <!-- Customer List -->
            <div id="customerList" class="list">
                <c:if test="${not empty sessionScope.selectedColumns}">
                    <c:forEach var="selectedColumn" items="${sessionScope.selectedColumns}">
                        <label onclick="selectLabel(this)">
                            <bean:write name="selectedColumn" />
                        </label>
                    </c:forEach>
                </c:if>
            </div>
        </div>

        <!-- Hidden input to store the selected columns -->
        <input type="hidden" name="selectedColumns" id="selectedColumns" 
               value="<%= request.getSession().getAttribute("selectedColumns") != null ? 
                       String.join(",", (String[]) request.getSession().getAttribute("selectedColumns")) : "conchonay" %>" />

        <br><br>
        <button type="submit" onclick="saveSelectedColumns()" name=action value="save">save</button>
    </html:form>
</body>
</html>

```
```
setting action

package fjs.cs.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.dao.SearchDao;

public class UploadAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        // Danh sách tất cả các cột
    	HttpSession session = request.getSession();
        String[] allColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"};
        String selectedColumns = request.getParameter("selectedColumns");
        String action = request.getParameter("action");
        String[] avaiDefault = (String[]) session.getAttribute("availableColumns");
        if(avaiDefault==null) {
            List<String> defaultColumns = Arrays.asList("Customer ID", "Customer Name", "Sex", "Birthday", "Address");

            // Các cột còn lại (bao gồm "Email") sẽ là cột chưa được chọn
            List<String> availableList = new ArrayList<>();
            for (String column : allColumns) {
                if (!defaultColumns.contains(column)) {
                    availableList.add(column);
                }
            }

            // Lưu danh sách cột chưa được chọn vào session
            session.setAttribute("availableColumns", availableList.toArray(new String[0]));
        }
        List<String> selectedList = new ArrayList<>();
        List<String> availableList = new ArrayList<>();
        if("save".equals(action)) {
            // Chuyển danh sách cột đã chọn thành danh sách
            if (selectedColumns != null && !selectedColumns.isEmpty()) {
                selectedList = Arrays.asList(selectedColumns.split(","));
                // Lưu danh sách cột đã chọn vào session
                request.getSession().setAttribute("selectedColumns", selectedList.toArray(new String[0]));
            }

            // Các cột chưa được chọn (cột ẩn) = Tất cả các cột - Cột đã chọn
            for (String column : allColumns) {
                if (!selectedList.contains(column)) {
                    availableList.add(column);
                    System.out.println(column);
                }
            }

            // Lưu danh sách cột ẩn vào session
            request.getSession().setAttribute("availableColumns", availableList.toArray(new String[0]));
            // Chuyển đến trang JSP setting header
            return mapping.findForward("success");
        }
//        else {
//            // Nếu action không phải là "save", thiết lập availableColumns là mảng rỗng
//            session.setAttribute("availableColumns", new String[0]);
//        }
        return mapping.findForward("success");
    }
}

```
------ new------
```
chua test
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://struts.apache.org/tags-bean" prefix="bean" %>
<%@ taglib uri="http://struts.apache.org/tags-logic" prefix="logic" %>

<html:form action="/updateColumnSettings">
  <div class="settings-container">
    <!-- Label cho các cột đang hiển thị -->
    <div class="column-group">
      <h4>Columns Showing:</h4>
      <html:select property="selectedColumns" multiple="true" size="5" styleClass="column-select">
        <html:options collection="availableColumns" property="value" labelProperty="label"/>
      </html:select>
    </div>

    <!-- Nút chuyển -->
    <div class="transfer-buttons">
      <button type="button" onclick="moveColumns('right')">&gt;&gt;</button>
      <button type="button" onclick="moveColumns('left')">&lt;&lt;</button>
    </div>

    <!-- Label cho các cột bị ẩn -->
    <div class="column-group">
      <h4>Hidden Columns:</h4>
      <html:select property="hiddenColumns" multiple="true" size="5" styleClass="column-select">
        <html:options collection="hiddenColumns" property="value" labelProperty="label"/>
      </html:select>
    </div>
  </div>

  <!-- Table hiển thị dữ liệu -->
  <table id="dataTable">
    <thead>
      <tr>
        <logic:iterate id="column" name="selectedColumns">
          <th class="column-${column.value}"><bean:write name="column" property="label"/></th>
        </logic:iterate>
      </tr>
    </thead>
    <tbody>
      <logic:iterate id="row" name="dataList">
        <tr>
          <logic:iterate id="column" name="selectedColumns">
            <td class="column-${column.value}">
              <bean:write name="row" property="${column.value}"/>
            </td>
          </logic:iterate>
        </tr>
      </logic:iterate>
    </tbody>
  </table>
</html:form>
```
```
.settings-container {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.column-group {
  flex: 1;
}

.column-select {
  width: 200px;
  height: 150px;
}

.transfer-buttons {
  display: flex;
  flex-direction: column;
  padding: 0 20px;
}

.transfer-buttons button {
  margin: 5px;
  padding: 5px 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
```
```
function moveColumns(direction) {
  var sourceSelect = direction === 'right' ? 
    document.getElementsByName('selectedColumns')[0] : 
    document.getElementsByName('hiddenColumns')[0];
  
  var targetSelect = direction === 'right' ? 
    document.getElementsByName('hiddenColumns')[0] : 
    document.getElementsByName('selectedColumns')[0];

  // Di chuyển các options đã chọn
  for(var i = sourceSelect.options.length - 1; i >= 0; i--) {
    var option = sourceSelect.options[i];
    if(option.selected) {
      targetSelect.add(option);
      updateTableColumns();
    }
  }
}

function updateTableColumns() {
  // Lấy danh sách các cột đang được chọn
  var selectedColumns = document.getElementsByName('selectedColumns')[0].options;
  var columnNames = Array.from(selectedColumns).map(opt => opt.value);
  
  // Ẩn/hiện các cột tương ứng
  var table = document.getElementById('dataTable');
  var headers = table.getElementsByTagName('th');
  var cells = table.getElementsByTagName('td');
  
  // Cập nhật hiển thị của các cột
  Array.from(headers).concat(Array.from(cells)).forEach(cell => {
    var columnClass = cell.className;
    var columnName = columnClass.replace('column-', '');
    cell.style.display = columnNames.includes(columnName) ? '' : 'none';
  });
}

// Tự động lưu settings khi có thay đổi
function saveSettings() {
  var selectedColumns = Array.from(document.getElementsByName('selectedColumns')[0].options)
    .map(opt => opt.value);
  
  // Gọi AJAX để lưu settings
  var xhr = new XMLHttpRequest();
  xhr.open('POST', 'saveColumnSettings.do', true);
  xhr.setRequestHeader('Content-Type', 'application/json');
  xhr.send(JSON.stringify({ columns: selectedColumns }));
}
```
```
public class ColumnSettingsAction extends Action {
    
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        
        // Khởi tạo danh sách các cột có sẵn
        List<LabelValueBean> availableColumns = new ArrayList<>();
        availableColumns.add(new LabelValueBean("ID", "id"));
        availableColumns.add(new LabelValueBean("Name", "name"));
        availableColumns.add(new LabelValueBean("Sex", "sex"));
        availableColumns.add(new LabelValueBean("Email", "email"));
        availableColumns.add(new LabelValueBean("Address", "address"));
        
        // Lấy settings từ database hoặc session
        UserSettings settings = getUserSettings(request);
        
        // Phân chia cột hiện/ẩn dựa trên settings
        List<LabelValueBean> selectedColumns = new ArrayList<>();
        List<LabelValueBean> hiddenColumns = new ArrayList<>();
        
        for(LabelValueBean column : availableColumns) {
            if(settings.isColumnVisible(column.getValue())) {
                selectedColumns.add(column);
            } else {
                hiddenColumns.add(column);
            }
        }
        
        // Set attributes
        request.setAttribute("availableColumns", selectedColumns);
        request.setAttribute("hiddenColumns", hiddenColumns);
        
        return mapping.findForward("success");
    }
}
```
```
public class ColumnSettingsForm extends ActionForm {
    private String[] selectedColumns;
    private String[] hiddenColumns;
    
    // Getters and setters
    public String[] getSelectedColumns() {
        return selectedColumns;
    }
    
    public void setSelectedColumns(String[] selectedColumns) {
        this.selectedColumns = selectedColumns;
    }
    
    public String[] getHiddenColumns() {
        return hiddenColumns;
    }
    
    public void setHiddenColumns(String[] hiddenColumns) {
        this.hiddenColumns = hiddenColumns;
    }
}
```
neeeeee
```
upload action
package fjs.cs.action;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.apache.struts.upload.FormFile;

import fjs.cs.form.UploadForm;
import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;

public class UploadAction extends Action {
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        UploadForm uploadForm = (UploadForm) form;
        FormFile file = uploadForm.getFile();
        String action = request.getParameter("action");

		// Kiểm tra khi ấn nút "Upload"
		if ("Upload".equals(action)) {
		// Kiểm tra nếu không có file được chọn
		if (file == null || file.getFileSize() == 0) {
		ActionMessages errors = new ActionMessages();
		errors.add("file", new ActionMessage("error.file.notexist"));
		saveErrors(request, errors);
		return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
		}
		
		// Đặt fileName vào form nếu file hợp lệ
		String fileName = file.getFileName();
		uploadForm.setFileName(fileName);
		ActionMessages errors = new ActionMessages();
		List<CustomerDto> customers = readCustomersFromFile(file, errors);
		
		// Kiểm tra nếu có lỗi
		if (!errors.isEmpty()) {
		saveErrors(request, errors);
		return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
		}
		
		// Biến lưu index của các dòng được insert và update
		List<Integer> insertedLines = new ArrayList<>();
		List<Integer> updatedLines = new ArrayList<>();
		
		// Thêm hoặc cập nhật từng customer trong danh sách
		int lineNumber = 1;
		for (CustomerDto customer : customers) {
		if ("male".equalsIgnoreCase(customer.getSex())) {
		   customer.setSex("0"); // Male thành 0
		} else if ("female".equalsIgnoreCase(customer.getSex())) {
		   customer.setSex("1"); // Female thành 1
		}
		
		// Kiểm tra nếu có customerId thì update, nếu không có thì add mới
		if (customer.getCustomerID() != 0) {
		   // Kiểm tra xem customer đã tồn tại chưa
		   if (isCustomerExists(customer.getCustomerID())) {
		       searchDao.editCustomer(customer);  // Gọi hàm editCustomer nếu đã tồn tại
		       updatedLines.add(lineNumber); // Thêm index dòng được update vào danh sách
		   } else {
		       searchDao.addCustomer(customer);   // Gọi hàm addCustomer nếu không tồn tại
		       insertedLines.add(lineNumber); // Thêm index dòng được insert vào danh sách
		   }
		} else {
		   // Nếu không có customerId thì thêm mới
		   searchDao.addCustomer(customer);
		   insertedLines.add(lineNumber); // Thêm index dòng được insert vào danh sách
		}
		lineNumber++;
		}
		
		// Tạo thông báo thành công
		ActionMessages successMessages = new ActionMessages();
		successMessages.add("success", new ActionMessage("message.success.general"));  // Thêm thông báo "Successfully"
		
		if (!insertedLines.isEmpty()) {
			System.out.println("no vo");
			successMessages.add("insertSuccess", 
		    new ActionMessage("message.success.insert", insertedLines.toString().replaceAll("[\\[\\]]", "")));
		}
		if (!updatedLines.isEmpty()) {
			System.out.println("no cung vo");
			successMessages.add("updateSuccess", 
		    new ActionMessage("message.success.update", updatedLines.toString().replaceAll("[\\[\\]]", "")));
		}
		
		// Lưu thông báo thành công vào request
		saveMessages(request, successMessages);
		
		// Tiến hành xử lý tiếp nếu file hợp lệ
		return mapping.findForward("success");
		}

        // Nếu không ấn nút "Upload", trả về trang hiện tại
        return mapping.findForward("success");
    }
    private List<CustomerDto> readCustomersFromFile(FormFile file, ActionMessages errors) {
        List<CustomerDto> customers = new ArrayList<>();
        InputStream inputStream = null;
        BufferedReader reader = null;

        try {
            inputStream = file.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream));

            String line;
            // Bỏ qua dòng tiêu đề
            reader.readLine();

            int lineNumber = 1; // Đếm số dòng để xác định dòng lỗi
            while ((line = reader.readLine()) != null) {
                String[] values = line.split(","); // Chia tách theo dấu phẩy

                // Kiểm tra độ dài mảng để đảm bảo không có giá trị thiếu
                if (values.length >= 6) {
                    String customerIdStr = values[0].trim();
                    String customerName = values[1].trim();
                    String sex = values[2].trim();
                    String birthday = values[3].trim();
                    String email = values[4].trim();
                    String address = values[5].trim();

                    CustomerDto customer = null;
                    boolean hasErrors = false; // Biến để theo dõi có lỗi hay không

                    // Kiểm tra nếu customerName là rỗng
                    if (customerName.isEmpty()) {
                        errors.add("customerName", new ActionMessage("error.customer.name.empty", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }
                    // Kiểm tra nếu email lớn hơn 40 ký tự
                    if (email.length() > 40) {
                        errors.add("email", new ActionMessage("error.email.tooLong", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }

                    // Nếu không có lỗi, kiểm tra customerId
                    if (!hasErrors) {
                        if (!customerIdStr.isEmpty()) {
                            int customerId = Integer.parseInt(customerIdStr);
                            // Kiểm tra sự tồn tại của customerId trong cơ sở dữ liệu
//                            if (!isCustomerExists(customerId)) {
//                                // Nếu deleteYmd khác null, thêm thông báo lỗi
//                                errors.add("customerIdNotExists", new ActionMessage("error.customer.not.exists", lineNumber, customerId));
//                                hasErrors = true; // Đánh dấu có lỗi
//                            } else {
                                customer = new CustomerDto(customerId, customerName, sex, birthday, email, address);
//                            }
                        } else {
                            // Tạo CustomerDto không bao gồm customerId
                            customer = new CustomerDto(customerName, sex, birthday, email, address);
                        }
                    }

                    // Nếu không có lỗi, thêm customer vào danh sách
                    if (!hasErrors && customer != null) {
                        customers.add(customer);
                    }
                }
                lineNumber++;
            }
        } catch (Exception e) {
            e.printStackTrace(); // Ghi log hoặc xử lý lỗi khác
        } finally {
            try {
                if (reader != null) {
                    reader.close();
                }
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (Exception e) {
                e.printStackTrace(); // Xử lý lỗi đóng file
            }
        }

        return customers;
    }

    private boolean isCustomerExists(int customerId) {
        // Phương thức kiểm tra xem customerId có tồn tại trong cơ sở dữ liệu hay không
        // Giả sử bạn đã có một lớp DAO để truy vấn dữ liệu
        return searchDao.isCustomerExists(customerId);
    }
}


```
```
message.success.general=Successfully \\n
message.success.insert=Insert line(s): {0} \\n
message.success.update=Update line(s): {0} \\n
```
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<html>
<head>
    <title>Upload File</title>
    <style>
        .container {
            display: flex;
            justify-content: space-between;
            width: 400px;
        }
        .hidden-file-input {
            display: none;
        }
        .custom-button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px;
            cursor: pointer;
        }
    </style>
    <script>
        function triggerFileInput() {
            document.getElementById('fileInput').click();
        }
        function updateFileName() {
            var input = document.getElementById('fileInput');
            var fileName = input.files[0] ? input.files[0].name : '';
            document.getElementById('fileName').value = fileName;
        }
        window.onload = function() {
            setTimeout(function() {
                var errorMessages = document.getElementById("htmlErrors").innerText.trim();
                if (errorMessages) {
                	alert(errorMessages.replace(/\\n/g, '\n'));
                }
            }, 100); // Thời gian chờ 2000ms (2 giây)
            // Hiển thị alert cho thành công
            setTimeout(function() {
                var successMessages = document.getElementById("messages").innerText.trim();
                if (successMessages) {
                	alert(successMessages.replace(/\\n/g, '\n'));
                }
            }, 100); // Thời gian chờ 2000ms (2 giây)
        }
    </script>
</head>
<body>
    <h2>Upload File Example</h2>
    <p id="messageParagraph">Hahahahea</p>
    <p id="testP"></p>
    <html:errors/>
    <h2>Example</h2>
    <!-- Thẻ chứa html:errors nhưng ẩn đi -->
    <div id="htmlErrors" style="display:none;">
        <html:errors />
    </div>
    <!-- Thẻ chứa thông báo thành công nhưng ẩn đi -->
    <c:out value="${successMessages}" />
    <div id= messages>
        <html:messages id="aMsg" message="true">
    		<bean:write name="aMsg" filter="false" />
    	</html:messages>
    </div>
    <html:form action="/upload" enctype="multipart/form-data" onsubmit="showParagraphContent();">
        <div class="container">
            <input type="text" id="fileName" name="fileName" value="<%= request.getAttribute("fileName") != null ? request.getAttribute("fileName").toString() : "" %>" readonly="true" />
            <input type="file" id="fileInput" class="hidden-file-input" name="file" onchange="updateFileName()" />
            <button type="button" class="custom-button" onclick="triggerFileInput()">Browse</button>
        </div>
        <br/>
        <button type="submit" value="Upload" name="action">Upload</button>
    </html:form>
</body>
</html>

```
https://drive.google.com/drive/folders/1bPh22WOHnfjV8E2gzLIvzQr1Bx1uL9nx?usp=sharing
```

uploadform
package fjs.cs.form;

import org.apache.struts.action.ActionForm;
import org.apache.struts.upload.FormFile;

public class UploadForm extends ActionForm {
    /**
	 * 
	 */
	private static final long serialVersionUID = 1L;
	private FormFile file;
    private String fileName;

    public FormFile getFile() {
        return file;
    }

    public void setFile(FormFile file) {
        this.file = file;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }
}

```
```
error.username.required=ChÆ°a nháº­p user name
error.password.required=ChÆ°a nháº­p password
error.login.invalid= ããã«ã¡ã¯
error.birthday.invalid=ngay sinh khong hop le
error.delete.noselection=Please select at least one customer to delete.
error.file.notexist=File import not existed
error.customer.name.empty = Line {0} : Customer name is empty \\n
error.email.tooLong = Line {0} : value email is more than 40
error.customer.not.exists=Line {0}: customerId = {1} is not existed
dao
package fjs.cs.dao;

import java.util.ArrayList;
import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import fjs.cs.dto.CustomerDto;
import fjs.cs.model.Customer;

public class SearchDao {
	
	private SessionFactory sessionFactory;

	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}
	@SuppressWarnings("unchecked")
	public List<CustomerDto> getAllCustomers() {
	    Session session = sessionFactory.openSession();
	    String hql = "SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
	    List<Object[]> results = session.createQuery(hql).list();
	    session.close();
	
	    List<CustomerDto> customers = new ArrayList<>();
	    for (Object[] row : results) {
	        CustomerDto dto = new CustomerDto();
	        dto.setCustomerID((int) row[0]);
	        dto.setCustomerName((String) row[1]);
	        dto.setSex((String) row[2]);
	        dto.setBirthday((String) row[3]);
	        dto.setAddress((String) row[4]);
	        customers.add(dto);
	    }
	
	    return customers;
	}		
//	@SuppressWarnings("unchecked")
//	public List<CustomerDto> getAllCustomers() {
//    	Transaction transaction = null;
//    	List<CustomerDto> customer = new ArrayList<>();
//        Session session = sessionFactory.openSession();
//        transaction = session.beginTransaction();
//        String hql = "SELECT new fjs.cs.dto.CustomerDto(c.customerID, c.customerName, c.sex, c.birthday, c.address) " +
//                "FROM Customer c WHERE c.delete_ymd IS NULL";
//        Query query = session.createQuery(hql);
//        customer  = query.list();
//        transaction.commit();
//        return customer;
//    }
	
    public List<Object[]> getAllCustomerNamesAndAddresses() {
        Transaction transaction = null;
        List<Object[]> customerList = null;
        Session session = sessionFactory.openSession();

        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Viết câu HQL để lấy customerName và address
            String hql = "SELECT c.customerName, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
            Query query = session.createQuery(hql);  // Sử dụng kiểu Query không generic

            // Lấy danh sách kết quả và đảm bảo nó là List<Object[]>
            customerList = (List<Object[]>) query.list(); // Cast kết quả về List<Object[]>
            
            // Commit transaction
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
        
        return customerList;
    }
	@SuppressWarnings("unchecked")
	public List<CustomerDto> searchCustomers(String customerName, String sex, String birthdayFrom, String birthdayTo) {

	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Tạo câu truy vấn HQL
	        StringBuilder hql = new StringBuilder("SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL");
	        List<Object> params = new ArrayList<>();

	        // Thêm điều kiện tìm kiếm
	        if (customerName != null && !customerName.trim().isEmpty()) {
	            hql.append(" AND c.customerName LIKE ?");
	            params.add("%" + customerName + "%");
	        }
	        
	        if (sex != null && !sex.trim().isEmpty()) {
	            hql.append(" AND c.sex = ?");
	            params.add(sex);
	        }
	        
	        if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
	            hql.append(" AND c.birthday >= ?");
	            params.add(birthdayFrom);
	        }
	        
	        if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
	            hql.append(" AND c.birthday <= ?");
	            params.add(birthdayTo);
	        }

	        // Thực thi query
	        Query query = session.createQuery(hql.toString());
	        for (int i = 0; i < params.size(); i++) {
	            query.setParameter(i, params.get(i));
	        }

	        // Lấy danh sách kết quả trả về
	        List<Object[]> results = query.list();  // Kết quả là danh sách Object[]
	        List<CustomerDto> customerDtos = new ArrayList<>();

	        // Xử lý kết quả trả về
	        for (Object[] row : results) {
	            CustomerDto dto = new CustomerDto(
	                (Integer) row[0],   // customerID
	                (String) row[1],    // customerName
	                (String) row[2],    // sex
	                (String) row[3],    // birthday
	                (String) row[4]     // address
	            );
	            customerDtos.add(dto);
	        }
	        
	        return customerDtos;
	    } finally {
	        session.close();
	    }
	}	
	public void addCustomer(CustomerDto customerDto) {
	    Transaction transaction = null;
	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Bắt đầu transaction
	        transaction = session.beginTransaction();

	        // Tạo đối tượng Customer từ CustomerDto
	        Customer customer = new Customer();
	        customer.setCustomerID(customerDto.getCustomerID()); // Nếu ID tự động sinh, bạn có thể bỏ qua dòng này
	        customer.setCustomerName(customerDto.getCustomerName());
	        customer.setSex(customerDto.getSex());
	        customer.setBirthday(customerDto.getBirthday());
	        customer.setAddress(customerDto.getAddress());
	        customer.setEmail(customerDto.getEmail()); // Nếu có email trong Dto
	        
	        // Lưu đối tượng Customer vào database
	        session.save(customer);
	        
	        // Commit transaction
	        transaction.commit();
	        
	    } catch (Exception e) {
	        if (transaction != null) {
	            transaction.rollback(); // Rollback nếu có lỗi
	        }
	        e.printStackTrace();
	    } finally {
	        session.close(); // Đóng session sau khi hoàn thành
	    }
	}
    public boolean isCustomerExists(int customerId) {
    	Session session = sessionFactory.openSession();
        boolean exists = false;

        try {
            // HQL query để kiểm tra sự tồn tại của customerId
            String hql = "SELECT count(c.customerID) FROM Customer c WHERE c.customerID = :customerID AND c.delete_ymd IS NULL";
            Query query = session.createQuery(hql); // Không sử dụng kiểu tham số hóa ở đây
            query.setParameter("customerId", customerId);

            Long count = (Long) query.uniqueResult(); // Lấy kết quả duy nhất
            exists = (count != null && count > 0); // Nếu có kết quả và > 0, thì customerId tồn tại
        } catch (Exception e) {
            e.printStackTrace(); // Xử lý lỗi
        } finally {
            session.close(); // Đóng session
        }
        return exists;
    }
}

```
```
uploadaction
package fjs.cs.action;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.apache.struts.upload.FormFile;

import fjs.cs.form.UploadForm;
import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto

public class UploadAction extends Action {
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        UploadForm uploadForm = (UploadForm) form;
        FormFile file = uploadForm.getFile();
        String action = request.getParameter("action");

        // Kiểm tra khi ấn nút "Upload"
        if ("Upload".equals(action)) {
            // Kiểm tra nếu không có file được chọn
            if (file == null || file.getFileSize() == 0) {
                ActionMessages errors = new ActionMessages();
                errors.add("file", new ActionMessage("error.file.notexist"));
                saveErrors(request, errors);
                return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
            }

            // Đặt fileName vào form nếu file hợp lệ
            String fileName = file.getFileName();
            uploadForm.setFileName(fileName);
            ActionMessages errors = new ActionMessages();
            List<CustomerDto> customers = readCustomersFromFile(file,errors);
            // Kiểm tra nếu có lỗi
            if (!errors.isEmpty()) {
                saveErrors(request, errors);
                return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
            }
         // Thêm từng customer trong danh sách vào cơ sở dữ liệu
            for (CustomerDto customer : customers) {
                if ("male".equalsIgnoreCase(customer.getSex())) {
                    customer.setSex("0"); // Male thành 0
                } else if ("female".equalsIgnoreCase(customer.getSex())) {
                    customer.setSex("1"); // Female thành 1
                }
            	searchDao.addCustomer(customer);  // Gọi hàm addCustomer cho từng customer
            }
            // Tiến hành xử lý tiếp nếu file hợp lệ
            return mapping.findForward("search");
        }

        // Nếu không ấn nút "Upload", trả về trang hiện tại
        return mapping.findForward("success");
    }
    private List<CustomerDto> readCustomersFromFile(FormFile file, ActionMessages errors) {
        List<CustomerDto> customers = new ArrayList<>();
        InputStream inputStream = null;
        BufferedReader reader = null;

        try {
            inputStream = file.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream));

            String line;
            // Bỏ qua dòng tiêu đề
            reader.readLine();

            int lineNumber = 1; // Đếm số dòng để xác định dòng lỗi
            while ((line = reader.readLine()) != null) {
                String[] values = line.split(","); // Chia tách theo dấu phẩy

                // Kiểm tra độ dài mảng để đảm bảo không có giá trị thiếu
                if (values.length >= 6) {
                    String customerIdStr = values[0].trim();
                    String customerName = values[1].trim();
                    String sex = values[2].trim();
                    String birthday = values[3].trim();
                    String email = values[4].trim();
                    String address = values[5].trim();

                    CustomerDto customer = null;
                    boolean hasErrors = false; // Biến để theo dõi có lỗi hay không

                    // Kiểm tra nếu customerName là rỗng
                    if (customerName.isEmpty()) {
                        errors.add("customerName", new ActionMessage("error.customer.name.empty", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }
                    // Kiểm tra nếu email lớn hơn 40 ký tự
                    if (email.length() > 40) {
                        errors.add("email", new ActionMessage("error.email.tooLong", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }

                    // Nếu không có lỗi, kiểm tra customerId
                    if (!hasErrors) {
                        if (!customerIdStr.isEmpty()) {
                            int customerId = Integer.parseInt(customerIdStr);
                            // Kiểm tra sự tồn tại của customerId trong cơ sở dữ liệu
                            if (!isCustomerExists(customerId)) {
                                // Nếu deleteYmd khác null, thêm thông báo lỗi
                                errors.add("customerIdNotExists", new ActionMessage("error.customer.not.exists", lineNumber, customerId));
                                hasErrors = true; // Đánh dấu có lỗi
                            } else {
                                customer = new CustomerDto(customerId, customerName, sex, birthday, email, address);
                            }
                        } else {
                            // Tạo CustomerDto không bao gồm customerId
                            customer = new CustomerDto(customerName, sex, birthday, email, address);
                        }
                    }

                    // Nếu không có lỗi, thêm customer vào danh sách
                    if (!hasErrors && customer != null) {
                        customers.add(customer);
                    }
                }
                lineNumber++;
            }
        } catch (Exception e) {
            e.printStackTrace(); // Ghi log hoặc xử lý lỗi khác
        } finally {
            try {
                if (reader != null) {
                    reader.close();
                }
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (Exception e) {
                e.printStackTrace(); // Xử lý lỗi đóng file
            }
        }

        return customers;
    }

    private boolean isCustomerExists(int customerId) {
        // Phương thức kiểm tra xem customerId có tồn tại trong cơ sở dữ liệu hay không
        // Giả sử bạn đã có một lớp DAO để truy vấn dữ liệu
        return searchDao.isCustomerExists(customerId);
    }
}


```
```
upload.jsp
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <title>Upload File</title>
    <style>
        .container {
            display: flex;
            justify-content: space-between;
            width: 400px;
        }
        .hidden-file-input {
            display: none;
        }
        .custom-button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px;
            cursor: pointer;
        }
    </style>
    <script>
        function triggerFileInput() {
            document.getElementById('fileInput').click();
        }
        function updateFileName() {
            var input = document.getElementById('fileInput');
            var fileName = input.files[0] ? input.files[0].name : '';
            document.getElementById('fileName').value = fileName;
        }
        window.onload = function() {
            setTimeout(function() {
                var errorMessages = document.getElementById("htmlErrors").innerText.trim();
                if (errorMessages) {
                	alert(errorMessages.replace(/\\n/g, '\n'));
                }
            }, 100); // Thời gian chờ 2000ms (2 giây)
        }
    </script>
</head>
<body>
    <h2>Upload File Example</h2>
    <p id="messageParagraph">Hahahahea</p>
    <p id="testP"></p>
    <html:errors/>
    <h2>Example</h2>
    <!-- Thẻ chứa html:errors nhưng ẩn đi -->
    <div id="htmlErrors" style="display:none;">
        <html:errors />
    </div>
    <html:form action="/upload" enctype="multipart/form-data" onsubmit="showParagraphContent(); return false;">
        <div class="container">
            <input type="text" id="fileName" name="fileName" value="<%= request.getAttribute("fileName") != null ? request.getAttribute("fileName").toString() : "" %>" readonly="true" />
            <input type="file" id="fileInput" class="hidden-file-input" name="file" onchange="updateFileName()" />
            <button type="button" class="custom-button" onclick="triggerFileInput()">Browse</button>
        </div>
        <br/>
        <button type="submit" value="Upload" name="action">Upload</button>
    </html:form>
</body>
</html>

```
-----------------------------------------------updatene--------------------------------------------------
```
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
	public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
    	HttpSession session = request.getSession();
    	SearchForm searchForm = (SearchForm) form;
    	String customerName = searchForm.getCustomerName();
        String sex = searchForm.getSex();
        String fromBirthday = searchForm.getBirthdayFrom();
        String toBirthday = searchForm.getBirthdayTo();
        ActionMessages errors = new ActionMessages();
        
        String action = request.getParameter("action");
    	List<CustomerDto> customerList;
//    	if ("export".equals(action)) {
//    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
//    	    
//    	    // Tạo file CSV
//    	    StringBuilder csvData = new StringBuilder();
//    	    
//    	    // Tiêu đề của các cột
//    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
//    	    
//    	    // Thêm dữ liệu khách hàng vào CSV, với các giá trị được bao quanh bởi dấu ngoặc kép
//    	    for (CustomerDto customer : customerList) {
//    	        csvData.append("\"").append(customer.getCustomerID()).append("\"").append(",")
//    	               .append("\"").append(customer.getCustomerName()).append("\"").append(",")
//    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\"").append(",")
//    	               .append("\"").append(customer.getBirthday()).append("\"").append(",")
//    	               .append("\"").append(customer.getAddress()).append("\"").append(",")
//    	               .append("\"").append(customer.getEmail()).append("\"").append("\n");
//    	    }
//    	    
//    	    // Đường dẫn đến nơi lưu file CSV
//    	    String filePath = "C:/Users/ASUS/Desktop/Data/customer_export.csv";
//    	    
//    	    try {
//    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
//    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
//    	        System.out.println("File exported successfully to " + filePath);
//    	    } catch (Exception e) {
//    	        e.printStackTrace();
//    	    }
//    	    
//    	    // Chuyển hướng hoặc thông báo thành công (có thể thêm redirect hoặc thông báo trong giao diện)
//    	    return mapping.findForward("search");
//    	}
    	if ("export".equals(action)) {
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Tạo file CSV
    	    StringBuilder csvData = new StringBuilder();
    	    
    	    // Tiêu đề của các cột
    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
    	    
    	    // Thêm dữ liệu khách hàng vào CSV
    	    for (CustomerDto customer : customerList) {
    	        csvData.append("\"").append(customer.getCustomerID()).append("\",")
    	               .append("\"").append(customer.getCustomerName()).append("\",")
    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
    	               .append("\"").append(customer.getBirthday()).append("\",")
    	               .append("\"").append(customer.getAddress()).append("\",")
    	               .append("\"").append(customer.getEmail()).append("\"\n");
    	    }
    	    
    	    // Đường dẫn đến nơi lưu file CSV
    	    String filePath = "C:/Users/ASUS/Desktop/Data/customer_export.csv";
    	    
    	    try {
    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
    	        System.out.println("File exported successfully to " + filePath);
    	    } catch (Exception e) {
    	        e.printStackTrace();
    	    }
    	    
    	    // Chuyển hướng hoặc thông báo thành công (có thể thêm redirect hoặc thông báo trong giao diện)
    	    return mapping.findForward("search");
    	}




    	if ("Search".equals(action)) {
    	    // Kiểm tra định dạng YYYY/MM/DD của birthdayFrom và birthdayTo
    	    String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";  // Regex kiểm tra định dạng YYYY/MM/DD}
    	    
    	    // Không có lỗi, thực hiện tìm kiếm
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Logic phân trang...
    	    int PAGE_SIZE = 2;
    	    String pageStr = request.getParameter("page");
    	    Integer currentPage = 1;
    	    if (pageStr != null) {
    	        currentPage = Integer.parseInt(pageStr);
    	    }
    	    if (currentPage == null || currentPage < 1) {
    	        currentPage = 1;
    	    }
    	    int startRow = (currentPage - 1) * PAGE_SIZE;
    	    int totalCustomers = customerList.size();
    	    int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
    	    
    	    List<CustomerDto> paginatedList = customerList.subList(
    	        Math.min(startRow, totalCustomers),
    	        Math.min(startRow + PAGE_SIZE, totalCustomers)
    	    );
    	    
    	    // Lưu thông tin tìm kiếm vào session
    	    session.setAttribute("customerName", customerName);
    	    session.setAttribute("sex", sex);
    	    session.setAttribute("fromBirthday", fromBirthday);
    	    session.setAttribute("toBirthday", toBirthday);
    	    
    	    session.setAttribute("customerList", paginatedList);
    	    session.setAttribute("currentPage", currentPage);
    	    session.setAttribute("totalPages", totalPages);
    	    
    	    return mapping.findForward("search");
    	}


        if (customerName != null || sex != null || fromBirthday != null || toBirthday != null) {
            customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
        } else {	
            customerList = searchDao.getAllCustomers();
        }
        int PAGE_SIZE = 2;
        String pageStr = request.getParameter("page");
        Integer currentPage = (Integer) session.getAttribute("currentPage");
        if (pageStr != null) {
            currentPage = Integer.parseInt(pageStr);
        }
        if (currentPage == null || currentPage < 1 ) {
            currentPage = 1;
        }
        int startRow = (currentPage - 1) * PAGE_SIZE;
        int totalCustomers = customerList.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
        List<CustomerDto> paginatedList = customerList.subList(
                Math.min(startRow, totalCustomers),
                Math.min(startRow + PAGE_SIZE, totalCustomers)
        );
        session.setAttribute("customerName", customerName);
        session.setAttribute("sex", sex);
        session.setAttribute("fromBirthday", fromBirthday);
        session.setAttribute("toBirthday", toBirthday);
        session.setAttribute("customerList", paginatedList);
        session.setAttribute("currentPage", currentPage);
        session.setAttribute("totalPages", totalPages);
    	return mapping.findForward("search");
    }
}

```
```
import fjs.cs.form.SearchForm;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.multipart.MultipartFile;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class SearchAction extends org.apache.struts.action.Action {
    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        SearchForm searchForm = (SearchForm) form;
        MultipartFile file = searchForm.getFile();

        // Kiểm tra file upload
        if (file == null || file.isEmpty()) {
            String alertMessage = "File import not exist";
            request.setAttribute("alertMessage", alertMessage);
            return mapping.findForward("failure");
        }

        // Kiểm tra định dạng file
        String fileName = file.getOriginalFilename();
        if (!fileName.endsWith(".csv")) {
            String alertMessage = "File import is not valid";
            request.setAttribute("alertMessage", alertMessage);
            return mapping.findForward("failure");
        }

        List<String> errorMessages = new ArrayList<>();

        try (BufferedReader reader = new BufferedReader(new InputStreamReader(file.getInputStream()))) {
            String line;
            int lineNumber = 1;

            // Bỏ qua tiêu đề
            reader.readLine();

            while ((line = reader.readLine()) != null) {
                String[] values = line.split(",");
                String customerId = values[0].trim(); // Giả sử customerId ở cột đầu tiên

                // Kiểm tra customerId không empty và không tồn tại trong bảng mstcustomer
                if (!customerId.isEmpty() && !isCustomerIdExists(customerId)) {
                    String errorMessage = String.format("Line %d: customerId = %s is not existed", lineNumber, customerId);
                    errorMessages.add(errorMessage);
                }
                lineNumber++;
            }
        } catch (Exception e) {
            // Xử lý lỗi đọc file
            String alertMessage = "Error reading file: " + e.getMessage();
            request.setAttribute("alertMessage", alertMessage);
            return mapping.findForward("failure");
        }

        // Nếu có lỗi, hiển thị tất cả các lỗi
        if (!errorMessages.isEmpty()) {
            StringBuilder allErrors = new StringBuilder();
            for (String errorMessage : errorMessages) {
                allErrors.append(errorMessage).append("\n");
            }
            request.setAttribute("alertMessage", allErrors.toString());
            return mapping.findForward("failure");
        }

        // Nếu không có lỗi, tiếp tục xử lý
        return mapping.findForward("success");
    }

    private boolean isCustomerIdExists(String customerId) {
        // Kiểm tra sự tồn tại của customerId trong bảng mstcustomer
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = null;
        boolean exists = false;

        try {
            transaction = session.beginTransaction();
            String hql = "FROM MstCustomer WHERE customerId = :customerId AND deleteYmd IS NULL";
            Query query = session.createQuery(hql);
            query.setParameter("customerId", customerId);
            exists = !query.list().isEmpty();
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) transaction.rollback();
            e.printStackTrace();
        } finally {
            session.close();
        }
        return exists;
    }
}

```
```
<c:if test="${not empty alertMessage}">
    <script>
        alert("${alertMessage}");
    </script>
</c:if>

```
```
ai
package fjs.cs.logic;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;

@Service
public class searchLogic {

    @Autowired
    private SearchDao searchDao;

    public List<CustomerDto> searchCustomers(String customerName, String sex, String fromBirthday, String toBirthday, int pageSize, int currentPage) {
        List<CustomerDto> allCustomers = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);

        // Logic phân trang
        int startRow = (currentPage - 1) * pageSize;
        int totalCustomers = allCustomers.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / pageSize);

        List<CustomerDto> paginatedList = allCustomers.subList(
            Math.min(startRow, totalCustomers),
            Math.min(startRow + pageSize, totalCustomers)
        );

        // Set additional attributes if needed
        // paginatedList.setTotalPages(totalPages);

        return paginatedList;
    }
}

ai
```
```
delete
import java.text.SimpleDateFormat;
import java.util.Date;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.Query;

public class CustomerDao {
    
    private SessionFactory sessionFactory;

    public void deleteCustomer(int customerID) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Lấy thời gian hiện tại
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            String currentDate = sdf.format(new Date());

            // Tạo câu lệnh HQL để cập nhật delete_ymd
            String hql = "UPDATE Customer SET delete_ymd = :delete_ymd WHERE customerID = :customerID";
            Query query = session.createQuery(hql);
            query.setParameter("delete_ymd", currentDate);
            query.setParameter("customerID", customerID);

            // Thực thi câu lệnh HQL
            int result = query.executeUpdate();

            // Commit transaction nếu không có lỗi
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}

```
```
add 
import org.hibernate.Session;
import org.hibernate.Transaction;

public class CustomerDao {
    
    private SessionFactory sessionFactory;

    public void addCustomer(CustomerDto customerDto) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();
            
            // Tạo đối tượng Customer từ CustomerDto
            Customer customer = new Customer();
            customer.setCustomerID(customerDto.getCustomerID());   // Nếu customerID là tự động thì bỏ qua dòng này
            customer.setCustomerName(customerDto.getCustomerName());
            customer.setSex(customerDto.getSex());
            customer.setBirthday(customerDto.getBirthday());
            customer.setAddress(customerDto.getAddress());
            customer.setDeleteYmd(null);  // Ban đầu giá trị delete_ymd là null
            
            // Lưu đối tượng vào cơ sở dữ liệu
            session.save(customer);

            // Commit transaction nếu không có lỗi
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}

```
```
edit 
import org.hibernate.Session;
import org.hibernate.Transaction;

public class CustomerDao {

    private SessionFactory sessionFactory;

    public void editCustomer(CustomerDto customerDto) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Truy xuất khách hàng từ database dựa trên customerID
            Customer customer = (Customer) session.get(Customer.class, customerDto.getCustomerID());
            
            // Kiểm tra nếu khách hàng tồn tại
            if (customer != null) {
                // Cập nhật thông tin từ customerDto
                customer.setCustomerName(customerDto.getCustomerName());
                customer.setSex(customerDto.getSex());
                customer.setBirthday(customerDto.getBirthday());
                customer.setAddress(customerDto.getAddress());
                // Không chỉnh sửa delete_ymd vì chỉ được thay đổi khi xóa
                
                // Cập nhật đối tượng trong database
                session.update(customer);

                // Commit transaction sau khi cập nhật
                transaction.commit();
            } else {
                System.out.println("Customer with ID " + customerDto.getCustomerID() + " not found.");
            }
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}

```
```
import org.hibernate.Session;
import org.hibernate.Transaction;
import java.util.List;

public class CustomerDao {

    private SessionFactory sessionFactory;

    @SuppressWarnings("unchecked")
    public List<Object[]> getAllCustomerNamesAndAddresses() {
        Transaction transaction = null;
        List<Object[]> customerList = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Viết câu HQL để lấy customerName và address
            String hql = "SELECT c.customerName, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
            Query query = session.createQuery(hql);
            
            // Thực thi query và lấy danh sách kết quả
            customerList = query.list();
            
            // Commit transaction
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
        
        return customerList;
    }
}

```

```
begin
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
	public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
    	HttpSession session = request.getSession();
    	SearchForm searchForm = (SearchForm) form;
    	String customerName = searchForm.getCustomerName();
        String sex = searchForm.getSex();
        String fromBirthday = searchForm.getBirthdayFrom();
        String toBirthday = searchForm.getBirthdayTo();
        ActionMessages errors = new ActionMessages();
        
        String action = request.getParameter("action");
    	List<CustomerDto> customerList;
    	if ("Search".equals(action)) {
    	    // Kiểm tra định dạng YYYY/MM/DD của birthdayFrom và birthdayTo
    	    String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";  // Regex kiểm tra định dạng YYYY/MM/DD}
    	    
    	    // Không có lỗi, thực hiện tìm kiếm
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Logic phân trang...
    	    int PAGE_SIZE = 2;
    	    String pageStr = request.getParameter("page");
    	    Integer currentPage = 1;
    	    if (pageStr != null) {
    	        currentPage = Integer.parseInt(pageStr);
    	    }
    	    if (currentPage == null || currentPage < 1) {
    	        currentPage = 1;
    	    }
    	    int startRow = (currentPage - 1) * PAGE_SIZE;
    	    int totalCustomers = customerList.size();
    	    int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
    	    
    	    List<CustomerDto> paginatedList = customerList.subList(
    	        Math.min(startRow, totalCustomers),
    	        Math.min(startRow + PAGE_SIZE, totalCustomers)
    	    );
    	    
    	    // Lưu thông tin tìm kiếm vào session
    	    session.setAttribute("customerName", customerName);
    	    session.setAttribute("sex", sex);
    	    session.setAttribute("fromBirthday", fromBirthday);
    	    session.setAttribute("toBirthday", toBirthday);
    	    
    	    session.setAttribute("customerList", paginatedList);
    	    session.setAttribute("currentPage", currentPage);
    	    session.setAttribute("totalPages", totalPages);
    	    
    	    return mapping.findForward("search");
    	}


        if (customerName != null || sex != null || fromBirthday != null || toBirthday != null) {
            customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
        } else {	
            customerList = searchDao.getAllCustomers();
        }
        int PAGE_SIZE = 2;
        String pageStr = request.getParameter("page");
        Integer currentPage = (Integer) session.getAttribute("currentPage");
        if (pageStr != null) {
            currentPage = Integer.parseInt(pageStr);
        }
        if (currentPage == null || currentPage < 1 ) {
            currentPage = 1;
        }
        int startRow = (currentPage - 1) * PAGE_SIZE;
        int totalCustomers = customerList.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
        List<CustomerDto> paginatedList = customerList.subList(
                Math.min(startRow, totalCustomers),
                Math.min(startRow + PAGE_SIZE, totalCustomers)
        );
        session.setAttribute("customerName", customerName);
        session.setAttribute("sex", sex);
        session.setAttribute("fromBirthday", fromBirthday);
        session.setAttribute("toBirthday", toBirthday);
        session.setAttribute("customerList", paginatedList);
        session.setAttribute("currentPage", currentPage);
        session.setAttribute("totalPages", totalPages);
    	return mapping.findForward("search");
    }
}

```
```
SEARCHDAO
package fjs.cs.dao;

import java.util.ArrayList;
import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import fjs.cs.dto.CustomerDto;
import fjs.cs.model.Customer;

public class SearchDao {
	
	private SessionFactory sessionFactory;


	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}
	@SuppressWarnings("unchecked")
	public List<CustomerDto> getAllCustomers() {
	    Session session = sessionFactory.openSession();
	    String hql = "SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
	    List<Object[]> results = session.createQuery(hql).list();
	    session.close();
	
	    List<CustomerDto> customers = new ArrayList<>();
	    for (Object[] row : results) {
	        CustomerDto dto = new CustomerDto();
	        dto.setCustomerID((int) row[0]);
	        dto.setCustomerName((String) row[1]);
	        dto.setSex((String) row[2]);
	        dto.setBirthday((String) row[3]);
	        dto.setAddress((String) row[4]);
	        customers.add(dto);
	    }
	
	    return customers;
	}		
//	@SuppressWarnings("unchecked")
//	public List<CustomerDto> getAllCustomers() {
//    	Transaction transaction = null;
//    	List<CustomerDto> customer = new ArrayList<>();
//        Session session = sessionFactory.openSession();
//        transaction = session.beginTransaction();
//        String hql = "SELECT new fjs.cs.dto.CustomerDto(c.customerID, c.customerName, c.sex, c.birthday, c.address) " +
//                "FROM Customer c WHERE c.delete_ymd IS NULL";
//        Query query = session.createQuery(hql);
//        customer  = query.list();
//        transaction.commit();
//        return customer;
//    }

	@SuppressWarnings("unchecked")
	public List<CustomerDto> searchCustomers(String customerName, String sex, String birthdayFrom, String birthdayTo) {

	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Tạo câu truy vấn HQL
	        StringBuilder hql = new StringBuilder("SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL");
	        List<Object> params = new ArrayList<>();

	        // Thêm điều kiện tìm kiếm
	        if (customerName != null && !customerName.trim().isEmpty()) {
	            hql.append(" AND c.customerName LIKE ?");
	            params.add("%" + customerName + "%");
	        }
	        
	        if (sex != null && !sex.trim().isEmpty()) {
	            hql.append(" AND c.sex = ?");
	            params.add(sex);
	        }
	        
	        if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
	            hql.append(" AND c.birthday >= ?");
	            params.add(birthdayFrom);
	        }
	        
	        if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
	            hql.append(" AND c.birthday <= ?");
	            params.add(birthdayTo);
	        }

	        // Thực thi query
	        Query query = session.createQuery(hql.toString());
	        for (int i = 0; i < params.size(); i++) {
	            query.setParameter(i, params.get(i));
	        }

	        // Lấy danh sách kết quả trả về
	        List<Object[]> results = query.list();  // Kết quả là danh sách Object[]
	        List<CustomerDto> customerDtos = new ArrayList<>();

	        // Xử lý kết quả trả về
	        for (Object[] row : results) {
	            CustomerDto dto = new CustomerDto(
	                (Integer) row[0],   // customerID
	                (String) row[1],    // customerName
	                (String) row[2],    // sex
	                (String) row[3],    // birthday
	                (String) row[4]     // address
	            );
	            customerDtos.add(dto);
	        }
	        
	        return customerDtos;
	    } finally {
	        session.close();
	    }
	}		
}

```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="fjs.cs.model.Customer" table="MSTCUSTOMERS">
	    <id name="customerID" column="CUSTOMERID" type="int">
	    <generator class="native"/>
	    </id>
		<property name="customerName" column="CUSTOMERNAME" type="string" length="50"/>
		<property name="sex" column="SEX" type="string" length="50"/>
		<property name="birthday" column="BIRTHDAY" type="string" length="50"/>
		<property name="address" column="ADDRESS" type="string" length="50"/>
		<property name="email" column="EMAIL" type="string" length="50"/>
		<property name="insert_psn" column="INSERT_PSN" type="int" length="50"/>
		<property name="update_psn" column="UPDATE_PSN" type="int" length="50"/>
		<property name="delete_ymd" column="DELETE_YMD" type="timestamp" />
		<property name="insert_ymd" column="INSERT_YMD" type="timestamp" />
		<property name="update_ymd" column="UPDATE_YMD" type="timestamp" />
	</class>
</hibernate-mapping>
```
```
package fjs.cs.model;

import java.sql.Date;

public class Customer {
	private int customerID;
	private String customerName;
	private String sex;
	private String birthday;
	private String address;
	private String email;
	private int insert_psn;
	private int update_psn;
	private Date delete_ymd;
	private Date insert_ymd;
	private Date update_ymd;
	
	public Customer() {
		super();
	}
	public Customer(int customerID, String customerName, String sex, String birthday, String address) {
		super();
		this.customerID = customerID;
		this.customerName = customerName;
		this.sex = sex;
		this.birthday = birthday;
		this.address = address;
	}
	public int getCustomerID() {
		return customerID;
	}
	public void setCustomerID(int customerID) {
		this.customerID = customerID;
	}
	public String getCustomerName() {
		return customerName;
	}
	public void setCustomerName(String customerName) {
		this.customerName = customerName;
	}
	public String getSex() {
		return sex;
	}
	public void setSex(String sex) {
		this.sex = sex;
	}
	public String getBirthday() {
		return birthday;
	}
	public void setBirthday(String birthday) {
		this.birthday = birthday;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public int getInsert_psn() {
		return insert_psn;
	}
	public void setInsert_psn(int insert_psn) {
		this.insert_psn = insert_psn;
	}
	public int getUpdate_psn() {
		return update_psn;
	}
	public void setUpdate_psn(int update_psn) {
		this.update_psn = update_psn;
	}
	public Date getDelete_ymd() {
		return delete_ymd;
	}
	public void setDelete_ymd(Date delete_ymd) {
		this.delete_ymd = delete_ymd;
	}
	public Date getInsert_ymd() {
		return insert_ymd;
	}
	public void setInsert_ymd(Date insert_ymd) {
		this.insert_ymd = insert_ymd;
	}
	public Date getUpdate_ymd() {
		return update_ymd;
	}
	public void setUpdate_ymd(Date update_ymd) {
		this.update_ymd = update_ymd;
	}
	
	
}

```
```
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
        <mapping resource="fjs/cs/model/Customer.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
```
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
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    	<property name="sessionFactory" ref="sessionFactory" />
	</bean>
    
    <bean id="loginDao" class="fjs.cs.dao.loginDao">
    	<property name="sessionFactory" ref="sessionFactory"/>
	</bean>
	<bean id="loginLogic" class="fjs.cs.logic.loginLogic">
    	<property name="loginDao" ref="loginDao"/>
	</bean>
	<bean name= "/login" id="loginAction" class="fjs.cs.action.loginAction">
	    <property name="loginLogic" ref="loginLogic"/>
	</bean>
	<bean id="searchDao" class="fjs.cs.dao.SearchDao">
    	<property name="sessionFactory" ref="sessionFactory"/>
	</bean>
	<bean id="searchLogic" class="fjs.cs.logic.SearchLogic">
    	<property name="searchDao" ref="searchDao"/>
	</bean>
	<bean name= "/search" id="SearchAction" class="fjs.cs.action.SearchAction">
		<property name="searchLogic" ref="searchLogic"/>
	</bean>
	
    
</beans>

```
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts-config PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN"
    "dtd/struts-config_1_3.dtd">

<struts-config>
	<form-beans>
	    <form-bean name="loginForm" type="fjs.cs.form.loginForm"/>
	    <form-bean name="searchForm" type="fjs.cs.form.SearchForm"/>
	</form-beans>
	
	<action-mappings>
	    <action path="/login"
	            type="org.springframework.web.struts.DelegatingActionProxy"
	            name="loginForm"
	            scope="request"
	            validate="false">
	      <forward name="success" path="/search.do" redirect="true"/>
	      <forward name="failure" path="/jsp/login.jsp" />
	    </action>
	    <action path="/search"
	            type="org.springframework.web.struts.DelegatingActionProxy"
	            name="searchForm"
	            scope="request"
	            validate="true">
	        <forward name="search" path="/jsp/Search.jsp" />
	        <forward name="login" path="/login.do" redirect="true"/>
	    </action>
	    
	</action-mappings>
	<controller processorClass="org.springframework.web.struts.DelegatingRequestProcessor"/>
	<message-resources parameter="message" />
</struts-config>

```
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" id="WebApp_ID" version="4.0">
  	<display-name>CustomerSystem_Struts</display-name>
	<context-param>
	    <param-name>javax.servlet.jsp.jstl.fmt.encoding</param-name>
	    <param-value>UTF-8</param-value>
	</context-param>
	    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
    </context-param>
    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

	<welcome-file-list>
	    <welcome-file>index.jsp</welcome-file> <!-- Đặt trang login.do làm welcome file -->
	</welcome-file-list>

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
</web-app>
```

DAY 5 --------------------------------------------


https://drive.google.com/drive/folders/1GCwNOsEnXPaDy0JoxNVx5eWhrDPhAVsL?usp=sharing
SQL SERVER
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình datasource kết nối SQL Server -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.microsoft.sqlserver.jdbc.SQLServerDriver" />
        <property name="url" value="jdbc:sqlserver://localhost:1433;databaseName=CustomerSystem"/>
        <property name="username" value="sa" />
        <property name="password" value="123456" />
    </bean>

    <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />
    </bean>

    <bean id="loginDao" class="fjs.cs.dao.loginDao">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

</beans>

```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.microsoft.sqlserver.jdbc.SQLServerDriver</property>
        <property name="hibernate.connection.url">jdbc:sqlserver://localhost:1433;databaseName=CustomerSystem</property>
        <property name="hibernate.connection.username">sa</property>
        <property name="hibernate.connection.password">123456</property>

        <!-- JDBC connection pool settings -->
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- Specify dialect for SQL Server -->
        <property name="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="hibernate.show_sql">true</property>

        <!-- Auto update schema -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Names the annotated entity class -->
        <mapping resource="fjs/cs/model/Login.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
SQL SERVER 
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
