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
