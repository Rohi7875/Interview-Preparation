# CodeIgniter Interview Questions & Answers
### For 7+ Years Experienced PHP Developers

---

## Table of Contents
1. [CodeIgniter Basics](#codeigniter-basics)
2. [MVC Architecture](#mvc-architecture)
3. [Controllers](#controllers)
4. [Models](#models)
5. [Views](#views)
6. [Database](#database)
7. [Libraries & Helpers](#libraries--helpers)
8. [Security](#security)
9. [Session & Cookies](#session--cookies)
10. [File Upload & Email](#file-upload--email)

---

## CodeIgniter Basics

### Q1. What is CodeIgniter?
**Answer:** CodeIgniter is a powerful PHP framework with a small footprint, built for developers who need a simple and elegant toolkit to create full-featured web applications.

**Key Features:**
- Small footprint (about 2MB)
- Exceptional performance
- Strong security features
- MVC architecture
- Built-in libraries
- Simple and clear documentation

**Real-time Example:**
```php
// application/config/routes.php
$route['default_controller'] = 'welcome';
$route['404_override'] = '';
$route['translate_uri_dashes'] = FALSE;

// Custom routes
$route['products'] = 'product/index';
$route['products/(:num)'] = 'product/view/$1';
$route['products/category/(:any)'] = 'product/category/$1';

// RESTful routes
$route['api/users']['GET'] = 'api/users/index';
$route['api/users/(:num)']['GET'] = 'api/users/show/$1';
$route['api/users']['POST'] = 'api/users/create';
$route['api/users/(:num)']['PUT'] = 'api/users/update/$1';
$route['api/users/(:num)']['DELETE'] = 'api/users/delete/$1';
```

### Q2. Explain CodeIgniter architecture
**Answer:** CodeIgniter follows MVC (Model-View-Controller) architecture:
- **Model** - Handles database operations
- **View** - Presentation layer
- **Controller** - Business logic and coordinator

**Application Flow:**
1. Index.php serves as front controller
2. Router examines HTTP request
3. Controller receives request
4. Controller loads Model/View/Library/Helper
5. View is rendered
6. Response sent to browser

**Real-time Example:**
```php
// application/controllers/Blog.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class Blog extends CI_Controller {
    
    public function __construct()
    {
        parent::__construct();
        // Load model
        $this->load->model('blog_model');
        // Load libraries
        $this->load->library('pagination');
        // Load helpers
        $this->load->helper(['url', 'text', 'date']);
    }
    
    public function index($page = 1)
    {
        // Configuration
        $config['base_url'] = base_url('blog/index');
        $config['total_rows'] = $this->blog_model->count_posts();
        $config['per_page'] = 10;
        
        $this->pagination->initialize($config);
        
        // Get data from model
        $data['posts'] = $this->blog_model->get_posts(
            $config['per_page'], 
            ($page - 1) * $config['per_page']
        );
        $data['pagination'] = $this->pagination->create_links();
        
        // Load view with data
        $this->load->view('templates/header');
        $this->load->view('blog/index', $data);
        $this->load->view('templates/footer');
    }
}
```

### Q3. What are hooks in CodeIgniter?
**Answer:** Hooks provide a way to tap into and modify the inner workings of the framework without hacking the core files.

**Hook Points:**
- pre_system - Very early, before system execution
- pre_controller - Before controller is called
- post_controller_constructor - After controller is instantiated
- post_controller - After controller is fully executed
- display_override - Override final display
- cache_override - Custom caching mechanism
- post_system - After final rendered page is sent

**Real-time Example:**
```php
// application/config/hooks.php
$hook['post_controller_constructor'] = array(
    'class'    => 'Login_check',
    'function' => 'check_login',
    'filename' => 'Login_check.php',
    'filepath' => 'hooks',
    'params'   => array()
);

$hook['pre_controller'] = array(
    'class'    => 'Maintenance',
    'function' => 'check_maintenance_mode',
    'filename' => 'Maintenance.php',
    'filepath' => 'hooks'
);

// application/hooks/Login_check.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class Login_check {
    
    protected $CI;
    
    public function __construct()
    {
        $this->CI =& get_instance();
    }
    
    public function check_login()
    {
        $controller = $this->CI->router->fetch_class();
        $method = $this->CI->router->fetch_method();
        
        // List of controllers/methods that don't require authentication
        $public_pages = [
            'login', 'register', 'forgot_password'
        ];
        
        if (!in_array($controller, $public_pages)) {
            if (!$this->CI->session->userdata('user_id')) {
                redirect('login');
            }
        }
    }
}

// application/hooks/Maintenance.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class Maintenance {
    
    public function check_maintenance_mode()
    {
        $CI =& get_instance();
        
        // Check if maintenance mode is enabled
        if (file_exists(FCPATH . '.maintenance')) {
            // Allow admin access
            $allowed_ips = ['127.0.0.1', '::1'];
            
            if (!in_array($CI->input->ip_address(), $allowed_ips)) {
                $CI->output
                    ->set_status_header(503)
                    ->set_output('
                        <h1>Site Under Maintenance</h1>
                        <p>We are currently performing scheduled maintenance.</p>
                        <p>We should be back shortly. Thank you for your patience.</p>
                    ')
                    ->_display();
                exit;
            }
        }
    }
}
```

---

## Controllers

### Q4. How to create and use controllers in CodeIgniter?
**Answer:** Controllers are the heart of your application, as they determine how HTTP requests should be handled.

**Real-time Example:**
```php
// application/controllers/User.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class User extends CI_Controller {
    
    public function __construct()
    {
        parent::__construct();
        $this->load->model('user_model');
        $this->load->library(['form_validation', 'session', 'upload']);
        $this->load->helper(['url', 'form', 'security']);
    }
    
    // Public method - accessible via URL
    public function index()
    {
        $this->check_login();
        
        $data['users'] = $this->user_model->get_all_users();
        $this->load->view('user/index', $data);
    }
    
    public function profile($user_id = NULL)
    {
        $this->check_login();
        
        $user_id = $user_id ?: $this->session->userdata('user_id');
        
        $data['user'] = $this->user_model->get_user($user_id);
        
        if (empty($data['user'])) {
            show_404();
        }
        
        $this->load->view('user/profile', $data);
    }
    
    public function register()
    {
        if ($this->session->userdata('user_id')) {
            redirect('dashboard');
        }
        
        if ($this->input->post()) {
            // Form validation
            $this->form_validation->set_rules('username', 'Username', 
                'required|min_length[3]|max_length[20]|is_unique[users.username]');
            $this->form_validation->set_rules('email', 'Email', 
                'required|valid_email|is_unique[users.email]');
            $this->form_validation->set_rules('password', 'Password', 
                'required|min_length[8]');
            $this->form_validation->set_rules('password_confirm', 'Confirm Password', 
                'required|matches[password]');
            
            if ($this->form_validation->run() === TRUE) {
                $data = [
                    'username' => $this->input->post('username'),
                    'email' => $this->input->post('email'),
                    'password' => password_hash($this->input->post('password'), PASSWORD_BCRYPT),
                    'created_at' => date('Y-m-d H:i:s')
                ];
                
                $user_id = $this->user_model->create_user($data);
                
                if ($user_id) {
                    $this->session->set_flashdata('success', 'Registration successful! Please login.');
                    redirect('login');
                } else {
                    $this->session->set_flashdata('error', 'Registration failed. Please try again.');
                }
            }
        }
        
        $this->load->view('user/register');
    }
    
    public function login()
    {
        if ($this->session->userdata('user_id')) {
            redirect('dashboard');
        }
        
        if ($this->input->post()) {
            $this->form_validation->set_rules('email', 'Email', 'required|valid_email');
            $this->form_validation->set_rules('password', 'Password', 'required');
            
            if ($this->form_validation->run() === TRUE) {
                $email = $this->input->post('email');
                $password = $this->input->post('password');
                
                $user = $this->user_model->get_user_by_email($email);
                
                if ($user && password_verify($password, $user['password'])) {
                    // Set session data
                    $session_data = [
                        'user_id' => $user['id'],
                        'username' => $user['username'],
                        'email' => $user['email'],
                        'role' => $user['role'],
                        'logged_in' => TRUE
                    ];
                    
                    $this->session->set_userdata($session_data);
                    
                    // Update last login
                    $this->user_model->update_last_login($user['id']);
                    
                    redirect('dashboard');
                } else {
                    $this->session->set_flashdata('error', 'Invalid email or password.');
                }
            }
        }
        
        $this->load->view('user/login');
    }
    
    public function logout()
    {
        $this->session->sess_destroy();
        redirect('login');
    }
    
    public function update_profile()
    {
        $this->check_login();
        
        $user_id = $this->session->userdata('user_id');
        
        if ($this->input->post()) {
            $this->form_validation->set_rules('username', 'Username', 
                'required|min_length[3]|max_length[20]');
            $this->form_validation->set_rules('email', 'Email', 
                'required|valid_email');
            $this->form_validation->set_rules('phone', 'Phone', 
                'required|numeric|min_length[10]|max_length[15]');
            
            if ($this->form_validation->run() === TRUE) {
                $data = [
                    'username' => $this->input->post('username'),
                    'email' => $this->input->post('email'),
                    'phone' => $this->input->post('phone'),
                    'address' => $this->input->post('address'),
                    'updated_at' => date('Y-m-d H:i:s')
                ];
                
                // Handle profile image upload
                if ($_FILES['profile_image']['name']) {
                    $upload_result = $this->_upload_image();
                    
                    if ($upload_result['success']) {
                        $data['profile_image'] = $upload_result['file_name'];
                    } else {
                        $this->session->set_flashdata('error', $upload_result['error']);
                        redirect('user/profile');
                    }
                }
                
                if ($this->user_model->update_user($user_id, $data)) {
                    $this->session->set_flashdata('success', 'Profile updated successfully!');
                } else {
                    $this->session->set_flashdata('error', 'Failed to update profile.');
                }
                
                redirect('user/profile');
            }
        }
        
        $data['user'] = $this->user_model->get_user($user_id);
        $this->load->view('user/edit_profile', $data);
    }
    
    // Private method - not accessible via URL
    private function check_login()
    {
        if (!$this->session->userdata('logged_in')) {
            redirect('login');
        }
    }
    
    private function _upload_image()
    {
        $config['upload_path'] = './uploads/profiles/';
        $config['allowed_types'] = 'jpg|jpeg|png|gif';
        $config['max_size'] = 2048; // 2MB
        $config['encrypt_name'] = TRUE;
        
        $this->upload->initialize($config);
        
        if ($this->upload->do_upload('profile_image')) {
            return [
                'success' => TRUE,
                'file_name' => $this->upload->data('file_name')
            ];
        } else {
            return [
                'success' => FALSE,
                'error' => $this->upload->display_errors()
            ];
        }
    }
}
```

---

## Models

### Q5. How to work with Models in CodeIgniter?
**Answer:** Models are PHP classes designed to work with database information.

**Real-time Example:**
```php
// application/models/User_model.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class User_model extends CI_Model {
    
    protected $table = 'users';
    
    public function __construct()
    {
        parent::__construct();
        $this->load->database();
    }
    
    // Create
    public function create_user($data)
    {
        if ($this->db->insert($this->table, $data)) {
            return $this->db->insert_id();
        }
        return FALSE;
    }
    
    // Read
    public function get_user($user_id)
    {
        return $this->db->where('id', $user_id)
                        ->get($this->table)
                        ->row_array();
    }
    
    public function get_user_by_email($email)
    {
        return $this->db->where('email', $email)
                        ->get($this->table)
                        ->row_array();
    }
    
    public function get_all_users($limit = NULL, $offset = 0)
    {
        $this->db->select('id, username, email, role, created_at, last_login');
        $this->db->from($this->table);
        $this->db->order_by('created_at', 'DESC');
        
        if ($limit) {
            $this->db->limit($limit, $offset);
        }
        
        return $this->db->get()->result_array();
    }
    
    public function get_users_with_profile($limit = 10, $offset = 0)
    {
        $this->db->select('u.*, p.bio, p.avatar, p.phone');
        $this->db->from($this->table . ' u');
        $this->db->join('user_profiles p', 'p.user_id = u.id', 'left');
        $this->db->order_by('u.created_at', 'DESC');
        $this->db->limit($limit, $offset);
        
        return $this->db->get()->result_array();
    }
    
    // Update
    public function update_user($user_id, $data)
    {
        return $this->db->where('id', $user_id)
                        ->update($this->table, $data);
    }
    
    public function update_last_login($user_id)
    {
        return $this->db->where('id', $user_id)
                        ->update($this->table, [
                            'last_login' => date('Y-m-d H:i:s')
                        ]);
    }
    
    // Delete
    public function delete_user($user_id)
    {
        return $this->db->where('id', $user_id)
                        ->delete($this->table);
    }
    
    // Search
    public function search_users($keyword, $limit = 10, $offset = 0)
    {
        $this->db->like('username', $keyword);
        $this->db->or_like('email', $keyword);
        $this->db->limit($limit, $offset);
        
        return $this->db->get($this->table)->result_array();
    }
    
    // Count
    public function count_users()
    {
        return $this->db->count_all($this->table);
    }
    
    public function count_by_role($role)
    {
        return $this->db->where('role', $role)
                        ->count_all_results($this->table);
    }
    
    // Complex queries
    public function get_active_users_with_posts()
    {
        $sql = "
            SELECT u.*, COUNT(p.id) as post_count
            FROM users u
            LEFT JOIN posts p ON p.user_id = u.id
            WHERE u.status = 'active'
            GROUP BY u.id
            HAVING post_count > 0
            ORDER BY post_count DESC
        ";
        
        return $this->db->query($sql)->result_array();
    }
    
    // Transaction example
    public function transfer_points($from_user_id, $to_user_id, $points)
    {
        $this->db->trans_start();
        
        // Deduct from sender
        $this->db->set('points', 'points - ' . $points, FALSE);
        $this->db->where('id', $from_user_id);
        $this->db->update($this->table);
        
        // Add to receiver
        $this->db->set('points', 'points + ' . $points, FALSE);
        $this->db->where('id', $to_user_id);
        $this->db->update($this->table);
        
        $this->db->trans_complete();
        
        return $this->db->trans_status();
    }
    
    // Validation
    public function email_exists($email, $exclude_id = NULL)
    {
        $this->db->where('email', $email);
        
        if ($exclude_id) {
            $this->db->where('id !=', $exclude_id);
        }
        
        return $this->db->count_all_results($this->table) > 0;
    }
}
```

This covers the core CodeIgniter concepts with practical examples. Would you like me to continue with more topics?

