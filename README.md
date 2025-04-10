<!DOCTYPE html>

<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EduSync Pro - 智能课程调度优化系统</title>

    <script src="https://cdn.socket.io/4.5.4/socket.io.min.js"></script>

    <!-- 预加载关键资源 -->
    <link rel="preload" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" as="style">
    <link rel="preload" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css" as="style">
    <link rel="preload" href="https://cdn.jsdelivr.net/npm/animate.css@4.1.1/animate.min.css" as="style">
    <link rel="preload" href="https://cdn.jsdelivr.net/npm/apexcharts@3.35.0/dist/apexcharts.min.js" as="script">

    <!-- 引入CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/animate.css@4.1.1/animate.min.css" rel="stylesheet">

    <!-- 添加主题颜色 -->
    <meta name="theme-color" content="#6f42c1">

    <style>
        :root {
            --primary-color: #6f42c1;
            --secondary-color: #6610f2;
            --dark-color: #343a40;
            --light-color: #f8f9fa;
            --success-color: #28a745;
            --danger-color: #dc3545;
            --warning-color: #ffc107;
            --info-color: #17a2b8;
            --transition-speed: 0.3s;
            --sidebar-width: 280px;
        }

        /* 暗黑模式变量 */
        @media (prefers-color-scheme: dark) {
            :root {
                --primary-color: #9d7feb;
                --secondary-color: #8a63d2;
                --dark-color: #212529;
                --light-color: #343a40;
            }

            body {
                background-color: #121212;
                color: #f8f9fa;
            }

            .card, .card-header {
                background-color: #2d2d2d;
                color: #f8f9fa;
            }

            .table {
                color: #f8f9fa;
            }

            .sidebar {
                background-color: #1a1a1a;
            }
        }

        /* 关键渲染路径优化 - 首屏样式内联 */
        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background-color: var(--light-color);
            color: #212529;
            margin: 0;
            padding: 0;
            overflow-x: hidden;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }

        /* 骨架屏动画 */
        @keyframes shimmer {
            0% { background-position: -468px 0 }
            100% { background-position: 468px 0 }
        }

        .skeleton {
            background: linear-gradient(to right, #f0f0f0 8%, #e0e0e0 18%, #f0f0f0 33%);
            background-size: 800px 104px;
            animation: shimmer 1.5s infinite linear;
            border-radius: 4px;
        }

        /* 平滑滚动 */
        html {
            scroll-behavior: smooth;
        }

        /* 侧边栏动画 */
        .sidebar {
            background-color: var(--dark-color);
            color: white;
            height: 100vh;
            position: fixed;
            width: var(--sidebar-width);
            transform: translateX(0);
            transition: transform var(--transition-speed) ease;
            z-index: 1000;
            box-shadow: 2px 0 10px rgba(0, 0, 0, 0.1);
        }

        .sidebar-collapsed {
            transform: translateX(calc(-1 * var(--sidebar-width) + 60px));
        }

        .sidebar .nav-link {
            color: rgba(255, 255, 255, 0.75);
            margin-bottom: 5px;
            border-radius: 5px;
            transition: all var(--transition-speed) ease;
            position: relative;
            overflow: hidden;
        }

        .sidebar .nav-link::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
            transition: all 0.6s ease;
        }

        .sidebar .nav-link:hover::before {
            left: 100%;
        }

        .sidebar .nav-link:hover, .sidebar .nav-link.active {
            color: white;
            background-color: rgba(255, 255, 255, 0.1);
            transform: translateX(5px);
        }

        .sidebar .nav-link i {
            margin-right: 10px;
            transition: transform var(--transition-speed) ease;
        }

        .sidebar .nav-link:hover i {
            transform: scale(1.2);
        }

        .main-content {
            margin-left: var(--sidebar-width);
            padding: 20px;
            transition: margin-left var(--transition-speed) ease;
            flex: 1;
        }

        .main-content-expanded {
            margin-left: 60px;
        }

        /* 卡片动画 */
        .card {
            border-radius: 12px;
            box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
            margin-bottom: 25px;
            border: none;
            transition: all 0.3s ease;
            overflow: hidden;
            background-color: white;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 12px 20px rgba(0, 0, 0, 0.15);
        }

        .card-header {
            background-color: var(--light-color);
            border-bottom: 1px solid rgba(0, 0, 0, 0.125);
            font-weight: 600;
            padding: 15px 20px;
            position: relative;
        }

        .card-header::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 3px;
            background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
            transform: scaleX(0);
            transform-origin: left;
            transition: transform 0.3s ease;
        }

        .card:hover .card-header::after {
            transform: scaleX(1);
        }

        /* 状态指示器 */
        .status-badge {
            font-size: 0.8rem;
            padding: 5px 10px;
            border-radius: 20px;
            transition: all 0.2s ease;
        }

        .status-badge:hover {
            transform: scale(1.05);
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }

        .progress {
            height: 10px;
            border-radius: 5px;
            overflow: hidden;
        }

        .progress-bar {
            transition: width 0.6s ease;
        }

        /* 教室状态 */
        .classroom-status {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
            padding: 8px;
            border-radius: 8px;
            transition: all 0.2s ease;
        }

        .classroom-status:hover {
            background-color: rgba(0, 0, 0, 0.05);
            transform: translateX(3px);
        }

        .status-indicator {
            width: 15px;
            height: 15px;
            border-radius: 50%;
            margin-right: 10px;
            transition: all 0.3s ease;
        }

        .classroom-status:hover .status-indicator {
            transform: scale(1.3);
        }

        .status-available {
            background-color: var(--success-color);
            box-shadow: 0 0 0 rgba(40, 167, 69, 0.4);
            animation: pulse-success 2s infinite;
        }

        .status-occupied {
            background-color: var(--danger-color);
            box-shadow: 0 0 0 rgba(220, 53, 69, 0.4);
            animation: pulse-danger 2s infinite;
        }

        .status-maintenance {
            background-color: var(--warning-color);
            box-shadow: 0 0 0 rgba(255, 193, 7, 0.4);
            animation: pulse-warning 2s infinite;
        }

        @keyframes pulse-success {
            0% { box-shadow: 0 0 0 0 rgba(40, 167, 69, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(40, 167, 69, 0); }
            100% { box-shadow: 0 0 0 0 rgba(40, 167, 69, 0); }
        }

        @keyframes pulse-danger {
            0% { box-shadow: 0 0 0 0 rgba(220, 53, 69, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(220, 53, 69, 0); }
            100% { box-shadow: 0 0 0 0 rgba(220, 53, 69, 0); }
        }

        @keyframes pulse-warning {
            0% { box-shadow: 0 0 0 0 rgba(255, 193, 7, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(255, 193, 7, 0); }
            100% { box-shadow: 0 0 0 0 rgba(255, 193, 7, 0); }
        }

        /* 课程表样式 */
        #timetable {
            font-size: 0.9rem;
            border-collapse: separate;
            border-spacing: 0;
        }

        #timetable th, #timetable td {
            text-align: center;
            vertical-align: middle;
            padding: 10px;
            position: relative;
        }

        #timetable th {
            background-color: var(--primary-color);
            color: white;
            position: sticky;
            top: 0;
            z-index: 10;
        }

        #timetable th:first-child {
            border-top-left-radius: 10px;
        }

        #timetable th:last-child {
            border-top-right-radius: 10px;
        }

        .time-slot {
            min-height: 70px;
            border: 1px solid #dee2e6;
            position: relative;
            transition: all 0.2s ease;
            background-color: white;
        }

        .time-slot:hover {
            background-color: rgba(111, 66, 193, 0.05);
            transform: scale(1.02);
            z-index: 2;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
        }

        .scheduled-class {
            background-color: var(--primary-color);
            color: white;
            border-radius: 6px;
            padding: 5px 8px;
            margin: 3px 0;
            font-size: 0.8rem;
            cursor: pointer;
            transition: all 0.2s ease;
            position: relative;
            overflow: hidden;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }

        .scheduled-class::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0));
            pointer-events: none;
        }

        .scheduled-class:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
        }

        .conflict {
            background-color: var(--danger-color);
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.02); }
            100% { transform: scale(1); }
        }

        /* 偏好匹配 */
        .preference-match {
            height: 12px;
            background-color: #e9ecef;
            border-radius: 6px;
            margin-top: 5px;
            overflow: hidden;
        }

        .preference-match-bar {
            height: 100%;
            border-radius: 6px;
            background: linear-gradient(90deg, var(--success-color), #5cb85c);
            transition: width 0.6s ease;
            position: relative;
        }

        .preference-match-bar::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: linear-gradient(90deg, rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0.3), rgba(255, 255, 255, 0.1));
            animation: shine 1.5s infinite;
        }

        @keyframes shine {
            0% { transform: translateX(-100%); }
            100% { transform: translateX(100%); }
        }

        /* 按钮样式 */
        .btn-primary {
            background-color: var(--primary-color);
            border-color: var(--primary-color);
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .btn-primary:hover {
            background-color: var(--secondary-color);
            border-color: var(--secondary-color);
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(111, 66, 193, 0.3);
        }

        .btn-primary:active {
            transform: translateY(0);
        }

        .btn-primary::after {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 5px;
            height: 5px;
            background: rgba(255, 255, 255, 0.5);
            opacity: 0;
            border-radius: 100%;
            transform: scale(1, 1) translate(-50%);
            transform-origin: 50% 50%;
        }

        .btn-primary:focus:not(:active)::after {
            animation: ripple 0.6s ease-out;
        }

        @keyframes ripple {
            0% {
                transform: scale(0, 0);
                opacity: 0.5;
            }
            100% {
                transform: scale(20, 20);
                opacity: 0;
            }
        }

        /* 表单元素 */
        .form-control, .form-select {
            border-radius: 8px;
            padding: 10px 15px;
            transition: all 0.3s ease;
            border: 1px solid #ced4da;
        }

        .form-control:focus, .form-select:focus {
            border-color: var(--primary-color);
            box-shadow: 0 0 0 0.25rem rgba(111, 66, 193, 0.25);
            transform: translateY(-1px);
        }

        /* 表格悬停效果 */
        .table-hover tbody tr {
            transition: all 0.2s ease;
        }

        .table-hover tbody tr:hover {
            background-color: rgba(111, 66, 193, 0.05);
            transform: translateX(3px);
        }

        /* 列表组项 */
        .list-group-item {
            transition: all 0.2s ease;
            border-radius: 8px !important;
            margin-bottom: 5px;
        }

        .list-group-item:hover {
            background-color: rgba(111, 66, 193, 0.1);
            transform: translateX(5px);
        }

        /* 响应式设计 */
        @media (max-width: 992px) {
            .sidebar {
                transform: translateX(-100%);
            }

            .sidebar.active {
                transform: translateX(0);
            }

            .main-content {
                margin-left: 0;
            }

            .navbar-toggler {
                display: block;
            }
        }

        /* 视差滚动效果 */
        .parallax-bg {
            background-attachment: fixed;
            background-position: center;
            background-repeat: no-repeat;
            background-size: cover;
            position: relative;
        }

        /* 骨架屏 */
        .skeleton-box {
            display: inline-block;
            height: 1em;
            position: relative;
            overflow: hidden;
            background-color: #e0e0e0;
            border-radius: 4px;
        }

        /* 加载动画 */
        .loader {
            display: inline-block;
            width: 30px;
            height: 30px;
            border: 3px solid rgba(111, 66, 193, 0.3);
            border-radius: 50%;
            border-top-color: var(--primary-color);
            animation: spin 1s ease-in-out infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        /* 自定义滚动条 */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }

        ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 10px;
        }

        ::-webkit-scrollbar-thumb {
            background: var(--primary-color);
            border-radius: 10px;
        }

        ::-webkit-scrollbar-thumb:hover {
            background: var(--secondary-color);
        }

        /* 模态框动画 */
        .modal.fade .modal-dialog {
            transform: translateY(-50px);
            opacity: 0;
            transition: all 0.3s ease;
        }

        .modal.show .modal-dialog {
            transform: translateY(0);
            opacity: 1;
        }

        /* 下拉菜单动画 */
        .dropdown-menu {
            display: block;
            opacity: 0;
            visibility: hidden;
            transform: translateY(10px);
            transition: all 0.3s ease;
        }

        .dropdown-menu.show {
            opacity: 1;
            visibility: visible;
            transform: translateY(0);
        }

        /* 工具提示动画 */
        .tooltip {
            transition: opacity 0.3s, transform 0.3s;
        }

        .tooltip.show {
            opacity: 1;
            transform: translateY(0);
        }

        /* 页面过渡动画 */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .page-transition {
            animation: fadeIn 0.5s ease forwards;
        }

        /* 主题切换按钮 */
        .theme-switcher {
            position: fixed;
            bottom: 20px;
            right: 20px;
            z-index: 1000;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background-color: var(--primary-color);
            color: white;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
            transition: all 0.3s ease;
        }

        .theme-switcher:hover {
            transform: scale(1.1) rotate(180deg);
            box-shadow: 0 6px 15px rgba(0, 0, 0, 0.3);
        }

        /* 响应式表格 */
        .table-responsive {
            overflow-x: auto;
            -webkit-overflow-scrolling: touch;
        }

        /* 拖放效果 */
        .draggable {
            cursor: move;
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }

        .draggable.dragging {
            opacity: 0.5;
            transform: scale(1.05);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
        }

        .dropzone {
            border: 2px dashed #ccc;
            border-radius: 8px;
            padding: 20px;
            text-align: center;
            transition: all 0.3s ease;
        }

        .dropzone.active {
            border-color: var(--primary-color);
            background-color: rgba(111, 66, 193, 0.05);
        }

        /* 骨架屏布局 */
        .skeleton-container {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }

        .skeleton-card {
            height: 150px;
            background: linear-gradient(to right, #f0f0f0 8%, #e0e0e0 18%, #f0f0f0 33%);
            background-size: 800px 104px;
            animation: shimmer 1.5s infinite linear;
            border-radius: 8px;
        }

        /* 可变字体 */
        @supports (font-variation-settings: normal) {
            body {
                font-family: 'Inter var', system-ui, -apple-system, sans-serif;
                font-variation-settings: 'wght' 400;
            }

            h1, h2, h3, h4, h5, h6 {
                font-variation-settings: 'wght' 700;
            }
        }

        /* 关键渲染路径优化 - 延迟加载样式 */
        .lazy-load {
            opacity: 0;
            transform: translateY(20px);
            transition: opacity 0.6s ease, transform 0.6s ease;
        }

        .lazy-load.loaded {
            opacity: 1;
            transform: translateY(0);
        }

        /* 打印样式 */
        @media print {
            .sidebar, .theme-switcher, .no-print {
                display: none !important;
            }

            .main-content {
                margin-left: 0 !important;
                padding: 0 !important;
            }

            .card {
                box-shadow: none !important;
                border: 1px solid #ddd !important;
                page-break-inside: avoid;
            }
    settings-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    transition: all 0.5s ease;
    transform: translateY(20px);
    opacity: 0;
}

.settings-container.active {
    transform: translateY(0);
    opacity: 1;
}

@media (max-width: 768px) {
    .settings-container {
        padding: 10px;
    }
}
        }
    </style>
</head>
<body>
    <!-- 移动端导航切换按钮 -->
    <button class="navbar-toggler d-lg-none position-fixed top-2 start-2 z-1050 bg-white rounded-circle p-2 shadow" id="sidebarToggle">
        <i class="bi bi-list"></i>
    </button>

    <!-- 主题切换按钮 -->
    <div class="theme-switcher" id="themeSwitcher">
        <i class="bi bi-moon-stars-fill"></i>
    </div>

    <div class="container-fluid">
        <div class="row">
            <!-- 侧边栏导航 -->
            <div class="col-md-2 sidebar p-0" id="sidebar">
                <div class="p-3">
                    <div class="d-flex justify-content-between align-items-center mb-4">
                        <h4 class="text-center mb-0 animate__animated animate__fadeIn">EduSync Pro</h4>
                        <button class="btn btn-sm btn-outline-light d-none d-lg-block" id="collapseSidebar">
                            <i class="bi bi-chevron-double-left"></i>
                        </button>
                    </div>
                    <ul class="nav flex-column">
                        <li class="nav-item">
                            <a class="nav-link active" href="#dashboard" data-bs-toggle="tab">
                                <i class="bi bi-speedometer2"></i> <span class="nav-text">控制面板</span>
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#scheduling" data-bs-toggle="tab">
                                <i class="bi bi-calendar-event"></i> <span class="nav-text">课程排班</span>
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#dynamic" data-bs-toggle="tab">
                                <i class="bi bi-arrow-repeat"></i> <span class="nav-text">动态调度</span>
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#preferences" data-bs-toggle="tab">
                                <i class="bi bi-heart"></i> <span class="nav-text">偏好匹配</span>
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#conflicts" data-bs-toggle="tab">
                                <i class="bi bi-exclamation-triangle"></i> <span class="nav-text">冲突检测</span>
                            </a>
                        </li>
                        <li class="nav-item ">
                            <a class="nav-link" href="#settings" data-bs-toggle="tab">
                                <i class="bi bi-gear"></i> <span class="nav-text">系统设置</span>
                            </a>
                        </li>
                    </ul>
                </div>
            </div>

            <!-- 主内容区 -->
            <div class="col-md-10 main-content" id="mainContent">
                <div class="tab-content">
                    <!-- 控制面板 -->
                    <div class="tab-pane fade show active page-transition" id="dashboard">
                        <h2 class="mb-4 animate__animated animate__fadeInDown">控制面板</h2>
                        <div class="row">
                            <div class="col-md-4 animate__animated animate__fadeInLeft">
                                <div class="card h-100">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>排课任务状态</span>
                                        <i class="bi bi-graph-up"></i>
                                    </div>
                                    <div class="card-body">
                                        <div id="scheduling-status">
                                            <p class="card-text">当前没有运行中的排课任务</p>
                                            <button id="start-scheduling" class="btn btn-primary">
                                                <span class="spinner-border spinner-border-sm d-none" id="schedulingSpinner"></span>
                                                开始排课
                                            </button>
                                        </div>
                                        <div id="task-progress" style="display: none;">
                                            <div class="d-flex justify-content-between mb-2">
                                                <span>遗传算法优化中...</span>
                                                <span id="progress-percent">0%</span>
                                            </div>
                                            <div class="progress">
                                                <div id="progress-bar" class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" style="width: 0%"></div>
                                            </div>
                                            <small class="text-muted">任务ID: <span id="task-id">-</span></small>
                                            <div class="mt-3" id="performance-metrics">
                                                <div class="d-flex justify-content-between small">
                                                    <span>CPU使用率:</span>
                                                    <span id="cpu-usage">0%</span>
                                                </div>
                                                <div class="d-flex justify-content-between small">
                                                    <span>内存使用:</span>
                                                    <span id="memory-usage">0 MB</span>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-4 animate__animated animate__fadeInUp">
                                <div class="card h-100">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>教室状态概览</span>
                                        <i class="bi bi-building"></i>
                                    </div>
                                    <div class="card-body">
                                        <div id="room-status-chart" style="height: 150px;"></div>
                                        <div class="classroom-status">
                                            <div class="status-indicator status-available"></div>
                                            <span>可用: <strong id="available-rooms">12</strong></span>
                                        </div>
                                        <div class="classroom-status">
                                            <div class="status-indicator status-occupied"></div>
                                            <span>占用中: <strong id="occupied-rooms">8</strong></span>
                                        </div>
                                        <div class="classroom-status">
                                            <div class="status-indicator status-maintenance"></div>
                                            <span>维护中: <strong id="maintenance-rooms">2</strong></span>
                                        </div>
                                        <div class="mt-3">
                                            <button class="btn btn-sm btn-outline-secondary" id="refresh-status">
                                                <i class="bi bi-arrow-clockwise"></i> 刷新状态
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-4 animate__animated animate__fadeInRight">
                                <div class="card h-100">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>系统通知</span>
                                        <i class="bi bi-bell"></i>
                                    </div>
                                    <div class="card-body p-0">
                                        <div class="list-group list-group-flush">
                                            <div class="alert alert-warning alert-dismissible fade show m-3" role="alert">
                                                <div class="d-flex align-items-center">
                                                    <i class="bi bi-exclamation-triangle-fill me-2"></i>
                                                    <div>
                                                        <strong>检测到冲突!</strong> 周三10:00-12:00有3个课程时间冲突。
                                                    </div>
                                                </div>
                                                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                                            </div>
                                            <div class="alert alert-info alert-dismissible fade show m-3" role="alert">
                                                <div class="d-flex align-items-center">
                                                    <i class="bi bi-info-circle-fill me-2"></i>
                                                    <div>
                                                        <strong>偏好调查完成率:</strong> 教师85%，学生72%。
                                                    </div>
                                                </div>
                                                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                                            </div>
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-check-circle text-success me-2"></i>
                                                    <span>排课任务 #1234 已完成</span>
                                                </div>
                                                <small class="text-muted">2分钟前</small>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-exclamation-triangle text-warning me-2"></i>
                                                    <span>教室 A201 设备需要维护</span>
                                                </div>
                                                <small class="text-muted">今天 10:30</small>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="row mt-4">
                            <div class="col-md-8 animate__animated animate__fadeInLeft">
                                <div class="card">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>本周课程表</span>
                                        <div>
                                            <button class="btn btn-sm btn-outline-primary me-2">
                                                <i class="bi bi-printer"></i> 打印
                                            </button>
                                            <button class="btn btn-sm btn-outline-secondary">
                                                <i class="bi bi-download"></i> 导出
                                            </button>
                                        </div>
                                    </div>
                                    <div class="card-body">
                                        <div class="table-responsive">
                                            <table class="table table-bordered" id="timetable">
                                                <thead>
                                                    <tr>
                                                        <th>时间</th>
                                                        <th>周一</th>
                                                        <th>周二</th>
                                                        <th>周三</th>
                                                        <th>周四</th>
                                                        <th>周五</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <!-- 动态生成课程表内容 -->
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-4 animate__animated animate__fadeInRight">
                                <div class="card h-100">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>资源使用率</span>
                                        <i class="bi bi-pie-chart"></i>
                                    </div>
                                    <div class="card-body">
                                        <div id="resource-usage-chart" style="height: 300px;"></div>
                                        <div class="mt-3 text-center">
                                            <button class="btn btn-sm btn-outline-primary" id="refresh-resource">
                                                <i class="bi bi-arrow-clockwise"></i> 刷新数据
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- 课程排班模块 -->
                    <div class="tab-pane fade page-transition" id="scheduling">
                        <h2 class="mb-4">课程排班</h2>
                        <div class="row">
                            <div class="col-md-8">
                                <div class="card animate__animated animate__fadeInLeft">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>排课设置</span>
                                        <i class="bi bi-sliders"></i>
                                    </div>
                                    <div class="card-body">
                                        <form id="scheduling-form">
                                            <div class="row mb-3">
                                                <div class="col-md-6">
                                                    <label for="algorithm-type" class="form-label">排课算法</label>
                                                    <select class="form-select" id="algorithm-type">
                                                        <option value="genetic">遗传算法</option>
                                                        <option value="constraint">约束满足</option>
                                                        <option value="hybrid">混合算法</option>
                                                    </select>
                                                </div>
                                                <div class="col-md-6">
                                                    <label for="max-iterations" class="form-label">最大迭代次数</label>
                                                    <input type="number" class="form-control" id="max-iterations" value="1000">
                                                </div>
                                            </div>
                                            <div class="row mb-3">
                                                <div class="col-md-6">
                                                    <label for="population-size" class="form-label">种群大小</label>
                                                    <input type="number" class="form-control" id="population-size" value="50">
                                                </div>
                                                <div class="col-md-6">
                                                    <label for="mutation-rate" class="form-label">变异率</label>
                                                    <div class="input-group">
                                                        <input type="range" class="form-range" id="mutation-rate-slider" min="0" max="1" step="0.01" value="0.01">
                                                        <input type="number" class="form-control" id="mutation-rate" value="0.01" step="0.01" min="0" max="1" style="width: 80px;">
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="mb-3">
                                                <label class="form-label">优先级设置</label>
                                                <div class="form-check form-switch mb-2">
                                                    <input class="form-check-input" type="checkbox" id="priority-teacher" checked>
                                                    <label class="form-check-label" for="priority-teacher">教师偏好优先</label>
                                                </div>
                                                <div class="form-check form-switch mb-2">
                                                    <input class="form-check-input" type="checkbox" id="priority-student" checked>
                                                    <label class="form-check-label" for="priority-student">学生偏好优先</label>
                                                </div>
                                                <div class="form-check form-switch">
                                                    <input class="form-check-input" type="checkbox" id="priority-room" checked>
                                                    <label class="form-check-label" for="priority-room">教室资源优化</label>
                                                </div>
                                            </div>
                                            <div class="d-grid gap-2">
                                                <button type="submit" class="btn btn-primary">
                                                    <i class="bi bi-play-circle me-2"></i> 开始排课
                                                </button>
                                            </div>
                                        </form>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-4">
                                <div class="card animate__animated animate__fadeInRight">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>排课历史</span>
                                        <i class="bi bi-clock-history"></i>
                                    </div>
                                    <div class="card-body p-0">
                                        <div class="list-group list-group-flush" id="scheduling-history">
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-check-circle-fill text-success me-2"></i>
                                                    <span>2023-06-15 排课</span>
                                                </div>
                                                <span class="badge bg-success rounded-pill">成功</span>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-check-circle-fill text-success me-2"></i>
                                                    <span>2023-06-10 排课</span>
                                                </div>
                                                <span class="badge bg-success rounded-pill">成功</span>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-exclamation-triangle-fill text-warning me-2"></i>
                                                    <span>2023-06-05 排课</span>
                                                </div>
                                                <span class="badge bg-warning rounded-pill">有冲突</span>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-check-circle-fill text-success me-2"></i>
                                                    <span>2023-05-28 排课</span>
                                                </div>
                                                <span class="badge bg-success rounded-pill">成功</span>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                                                <div>
                                                    <i class="bi bi-x-circle-fill text-danger me-2"></i>
                                                    <span>2023-05-20 排课</span>
                                                </div>
                                                <span class="badge bg-danger rounded-pill">失败</span>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                                <div class="card mt-4 animate__animated animate__fadeInRight animate__delay-1s">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>算法性能</span>
                                        <i class="bi bi-speedometer2"></i>
                                    </div>
                                    <div class="card-body">
                                        <div id="algorithm-performance-chart" style="height: 200px;"></div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- 动态调度模块 -->
                    <div class="tab-pane fade page-transition" id="dynamic">
                        <h2 class="mb-4">动态调度</h2>
                        <div class="row">
                            <div class="col-md-6">
                                <div class="card animate__animated animate__fadeInLeft">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>教室实时状态</span>
                                        <div>
                                            <button class="btn btn-sm btn-outline-secondary me-2" id="filter-available">
                                                <i class="bi bi-funnel"></i> 可用
                                            </button>
                                            <button class="btn btn-sm btn-outline-secondary" id="filter-all">
                                                全部
                                            </button>
                                        </div>
                                    </div>
                                    <div class="card-body">
                                        <div class="table-responsive">
                                            <table class="table table-hover" id="room-status-table">
                                                <thead>
                                                    <tr>
                                                        <th>教室编号</th>
                                                        <th>状态</th>
                                                        <th>当前课程</th>
                                                        <th>设备状态</th>
                                                        <th>操作</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <!-- 动态生成教室状态 -->
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="card animate__animated animate__fadeInRight">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>动态调整</span>
                                        <i class="bi bi-arrow-repeat"></i>
                                    </div>
                                    <div class="card-body">
                                        <form id="adjustment-form">
                                            <div class="mb-3">
                                                <label for="adjustment-reason" class="form-label">调整原因</label>
                                                <select class="form-select" id="adjustment-reason">
                                                    <option value="equipment">设备故障</option>
                                                    <option value="emergency">紧急情况</option>
                                                    <option value="preference">偏好变更</option>
                                                    <option value="other">其他</option>
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <label for="affected-rooms" class="form-label">受影响教室</label>
                                                <select class="form-select" id="affected-rooms" multiple>
                                                    <!-- 动态生成教室选项 -->
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <div class="input-group">
                                                    <input type="datetime-local" class="form-control" id="adjustment-start">
                                                    <span class="input-group-text">至</span>
                                                    <input type="datetime-local" class="form-control" id="adjustment-end">
                                                </div>
                                            </div>
                                            <div class="mb-3">
                                                <label for="adjustment-notes" class="form-label">备注</label>
                                                <textarea class="form-control" id="adjustment-notes" rows="3"></textarea>
                                            </div>
                                            <div class="d-grid gap-2">
                                                <button type="submit" class="btn btn-primary" id="submit-adjustment">
                                                    <i class="bi bi-send-check me-2"></i> 提交调整
                                                </button>
                                            </div>
                                        </form>
                                    </div>
                                </div>
                                <div class="card mt-4 animate__animated animate__fadeInRight animate__delay-1s">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>调整历史</span>
                                        <i class="bi bi-list-check"></i>
                                    </div>
                                    <div class="card-body p-0">
                                        <div class="list-group list-group-flush" id="adjustment-history">
                                            <a href="#" class="list-group-item list-group-item-action">
                                                <div class="d-flex justify-content-between">
                                                    <span>2023-06-15 10:00 时间冲突</span>
                                                    <span class="badge bg-success">已修复</span>
                                                </div>
                                                <small class="text-muted">自动调整到2023-06-15 14:00</small>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action">
                                                <div class="d-flex justify-content-between">
                                                    <span>2023-06-10 教室冲突</span>
                                                    <span class="badge bg-success">已修复</span>
                                                </div>
                                                <small class="text-muted">手动调整为A201教室</small>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action">
                                                <div class="d-flex justify-content-between">
                                                    <span>2023-06-05 设备故障</span>
                                                    <span class="badge bg-warning">处理中</span>
                                                </div>
                                                <small class="text-muted">教室B305投影仪维修</small>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- 偏好匹配模块 -->
                    <div class="tab-pane fade page-transition" id="preferences">
                        <h2 class="mb-4">偏好匹配</h2>
                        <div class="row">
                            <div class="col-md-6">
                                <div class="card animate__animated animate__fadeInLeft">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>教师偏好设置</span>
                                        <i class="bi bi-person-badge"></i>
                                    </div>
                                    <div class="card-body">
                                        <form id="teacher-pref-form">
                                            <div class="mb-3">
                                                <label for="teacher-select" class="form-label">选择教师</label>
                                                <select class="form-select" id="teacher-select">
                                                    <option value="">-- 选择教师 --</option>
                                                    <option value="1">张教授</option>
                                                    <option value="2">李教授</option>
                                                    <option value="3">王教授</option>
                                                    <option value="4">赵教授</option>
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <label class="form-label">时间偏好</label>
                                                <div class="btn-group w-100" role="group" aria-label="时间偏好">
                                                    <input type="checkbox" class="btn-check" id="teacher-morning" autocomplete="off">
                                                    <label class="btn btn-outline-primary" for="teacher-morning">上午</label>

                                                    <input type="checkbox" class="btn-check" id="teacher-afternoon" autocomplete="off">
                                                    <label class="btn btn-outline-primary" for="teacher-afternoon">下午</label>

                                                    <input type="checkbox" class="btn-check" id="teacher-evening" autocomplete="off">
                                                    <label class="btn btn-outline-primary" for="teacher-evening">晚上</label>
                                                </div>
                                            </div>
                                            <div class="mb-3">
                                                <label for="teacher-room-type" class="form-label">教室类型偏好</label>
                                                <select class="form-select" id="teacher-room-type" multiple>
                                                    <option value="lecture">讲座厅</option>
                                                    <option value="lab">实验室</option>
                                                    <option value="seminar">研讨室</option>
                                                    <option value="computer">计算机房</option>
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <label for="teacher-notes" class="form-label">其他要求</label>
                                                <textarea class="form-control" id="teacher-notes" rows="3"></textarea>
                                            </div>
                                            <div class="d-grid gap-2">
                                                <button type="submit" class="btn btn-primary" id="save-teacher-pref">
                                                    <i class="bi bi-save me-2"></i> 保存教师偏好
                                                </button>
                                            </div>
                                        </form>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="card animate__animated animate__fadeInRight">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>学生偏好设置</span>
                                        <i class="bi bi-people"></i>
                                    </div>
                                    <div class="card-body">
                                        <form id="student-pref-form">
                                            <div class="mb-3">
                                                <label for="student-group-select" class="form-label">选择学生组</label>
                                                <select class="form-select" id="student-group-select">
                                                    <option value="">-- 选择学生组 --</option>
                                                    <option value="1">计算机科学1班</option>
                                                    <option value="2">计算机科学2班</option>
                                                    <option value="3">信息管理1班</option>
                                                    <option value="4">软件工程1班</option>
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <label class="form-label">时间偏好</label>
                                                <div class="btn-group w-100" role="group" aria-label="时间偏好">
                                                    <input type="checkbox" class="btn-check" id="student-morning" autocomplete="off">
                                                    <label class="btn btn-outline-primary" for="student-morning">上午</label>

                                                    <input type="checkbox" class="btn-check" id="student-afternoon" autocomplete="off">
                                                    <label class="btn btn-outline-primary" for="student-afternoon">下午</label>

                                                    <input type="checkbox" class="btn-check" id="student-evening" autocomplete="off">
                                                    <label class="btn btn-outline-primary" for="student-evening">晚上</label>
                                                </div>
                                            </div>
                                            <div class="mb-3">
                                                <label for="student-teacher-pref" class="form-label">教师偏好</label>
                                                <select class="form-select" id="student-teacher-pref" multiple>
                                                    <option value="1">张教授</option>
                                                    <option value="2">李教授</option>
                                                    <option value="3">王教授</option>
                                                    <option value="4">赵教授</option>
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <label for="student-notes" class="form-label">其他要求</label>
                                                <textarea class="form-control" id="student-notes" rows="3"></textarea>
                                            </div>
                                            <div class="d-grid gap-2">
                                                <button type="submit" class="btn btn-primary" id="save-student-pref">
                                                    <i class="bi bi-save me-2"></i> 保存学生偏好
                                                </button>
                                            </div>
                                        </form>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="row mt-4">
                            <div class="col-md-12">
                                <div class="card animate__animated animate__fadeInUp">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>偏好匹配分析</span>
                                        <button class="btn btn-sm btn-primary" id="run-preference-match">
                                            <i class="bi bi-arrow-repeat me-2"></i> 运行匹配分析
                                        </button>
                                    </div>
                                    <div class="card-body">
                                        <div class="table-responsive">
                                            <table class="table table-hover" id="preference-match-table">
                                                <thead>
                                                    <tr>
                                                        <th>课程</th>
                                                        <th>教师</th>
                                                        <th>学生组</th>
                                                        <th>时间匹配度</th>
                                                        <th>教室匹配度</th>
                                                        <th>综合匹配度</th>
                                                        <th>冲突建议</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <!-- 动态生成匹配结果 -->
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- 冲突检测模块 -->
                    <div class="tab-pane fade page-transition" id="conflicts">
                        <h2 class="mb-4">冲突检测与修复</h2>
                        <div class="row">
                            <div class="col-md-6">
                                <div class="card animate__animated animate__fadeInLeft">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>冲突检测</span>
                                        <i class="bi bi-shield-exclamation"></i>
                                    </div>
                                    <div class="card-body">
                                        <form id="conflict-check-form">
                                            <div class="mb-3">
                                                <label for="conflict-check-type" class="form-label">检测类型</label>
                                                <select class="form-select" id="conflict-check-type">
                                                    <option value="all">全部检测</option>
                                                    <option value="time">时间冲突</option>
                                                    <option value="room">教室冲突</option>
                                                    <option value="teacher">教师冲突</option>
                                                    <option value="student">学生冲突</option>
                                                </select>
                                            </div>
                                            <div class="mb-3">
                                                <div class="input-group">
                                                    <input type="date" class="form-control" id="conflict-start-date">
                                                    <span class="input-group-text">至</span>
                                                    <input type="date" class="form-control" id="conflict-end-date">
                                                </div>
                                            </div>
                                            <div class="d-grid gap-2">
                                                <button type="submit" class="btn btn-primary" id="run-conflict-check">
                                                    <i class="bi bi-search me-2"></i> 运行冲突检测
                                                </button>
                                            </div>
                                        </form>
                                    </div>
                                </div>
                                <div class="card mt-4 animate__animated animate__fadeInLeft animate__delay-1s">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>检测结果</span>
                                        <span class="badge bg-danger" id="conflict-count">3 个冲突</span>
                                    </div>
                                    <div class="card-body p-0">
                                        <div class="table-responsive">
                                            <table class="table table-hover mb-0" id="conflict-result-table">
                                                <thead>
                                                    <tr>
                                                        <th>冲突类型</th>
                                                        <th>涉及对象</th>
                                                        <th>时间</th>
                                                        <th>教室</th>
                                                        <th>严重程度</th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <!-- 动态生成冲突结果 -->
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="card animate__animated animate__fadeInRight">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>冲突修复</span>
                                        <i class="bi bi-tools"></i>
                                    </div>
                                    <div class="card-body">
                                        <div class="mb-3">
                                            <label for="conflict-select" class="form-label">选择冲突</label>
                                            <select class="form-select" id="conflict-select">
                                                <option value="">-- 选择要修复的冲突 --</option>
                                                <option value="1">周三 10:00-12:00 时间冲突</option>
                                                <option value="2">周五 14:00-16:00 教室冲突</option>
                                                <option value="3">周二 8:00-10:00 学生冲突</option>
                                            </select>
                                        </div>
                                        <div class="mb-3">
                                            <label for="fix-strategy" class="form-label">修复策略</label>
                                            <select class="form-select" id="fix-strategy">
                                                <option value="auto">自动修复</option>
                                                <option value="manual">手动修复</option>
                                                <option value="suggest">建议方案</option>
                                            </select>
                                        </div>
                                        <div id="manual-fix-options" style="display: none;">
                                            <div class="mb-3">
                                                <label for="new-time" class="form-label">新时间</label>
                                                <input type="datetime-local" class="form-control" id="new-time">
                                            </div>
                                            <div class="mb-3">
                                                <label for="new-room" class="form-label">新教室</label>
                                                <select class="form-select" id="new-room">
                                                    <option value="">-- 选择教室 --</option>
                                                    <option value="A201">A201</option>
                                                    <option value="B305">B305</option>
                                                    <option value="C101">C101</option>
                                                    <option value="D402">D402</option>
                                                </select>
                                            </div>
                                        </div>
                                        <div id="suggestions-container" style="display: none;">
                                            <h6>建议方案</h6>
                                            <div class="list-group mb-3" id="fix-suggestions">
                                                <!-- 动态生成修复建议 -->
                                            </div>
                                        </div>
                                        <div class="d-grid gap-2">
                                            <button class="btn btn-primary" id="apply-fix">
                                                <i class="bi bi-check-circle me-2"></i> 应用修复
                                            </button>
                                        </div>
                                    </div>
                                </div>
                                <div class="card mt-4 animate__animated animate__fadeInRight animate__delay-1s">
                                    <div class="card-header d-flex justify-content-between align-items-center">
                                        <span>修复历史</span>
                                        <i class="bi bi-clock-history"></i>
                                    </div>
                                    <div class="card-body p-0">
                                        <div class="list-group list-group-flush" id="fix-history">
                                            <a href="#" class="list-group-item list-group-item-action">
                                                <div class="d-flex justify-content-between">
                                                    <span>2023-06-15 10:00 时间冲突</span>
                                                    <span class="badge bg-success">已修复</span>
                                                </div>
                                                <small class="text-muted">自动调整到2023-06-15 14:00</small>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action">
                                                <div class="d-flex justify-content-between">
                                                    <span>2023-06-10 教室冲突</span>
                                                    <span class="badge bg-success">已修复</span>
                                                </div>
                                                <small class="text-muted">手动调整为A201教室</small>
                                            </a>
                                            <a href="#" class="list-group-item list-group-item-action">
                                                <div class="d-flex justify-content-between">
                                                    <span>2023-06-05 教师冲突</span>
                                                    <span class="badge bg-success">已修复</span>
                                                </div>
                                                <small class="text-muted">调整教师为赵教授</small>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
                <!-- 系统设置模块 -->
<div class="tab-pane fade page-transition" id="settings">
    <div class="settings-container" style="max-width: 800px; margin-left: auto; margin-right: 10%;">
        <h2 class="mb-4 text-center">系统设置</h2>

        <!-- 账户设置 -->
        <div class="card animate__animated animate__fadeIn mb-4">
            <div class="card-header d-flex justify-content-between align-items-center">
                <span>账户设置</span>
                <i class="bi bi-person-gear"></i>
            </div>
            <div class="card-body">
                <form id="account-settings-form">
                    <div class="mb-3">
                        <label for="account-username" class="form-label">用户名</label>
                        <input type="text" class="form-control" id="account-username" value="admin" readonly>
                    </div>
                    <div class="mb-3">
                        <label for="account-email" class="form-label">邮箱地址</label>
                        <input type="email" class="form-control" id="account-email" value="admin@edusync.edu">
                    </div>
                    <div class="mb-3">
                        <label for="account-role" class="form-label">角色</label>
                        <select class="form-select" id="account-role">
                            <option value="admin">管理员</option>
                            <option value="scheduler">排课员</option>
                            <option value="viewer">查看者</option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label for="account-password" class="form-label">更改密码</label>
                        <input type="password" class="form-control" id="account-password" placeholder="输入新密码">
                    </div>
                    <div class="mb-3">
                        <label for="account-confirm-password" class="form-label">确认密码</label>
                        <input type="password" class="form-control" id="account-confirm-password" placeholder="再次输入新密码">
                    </div>
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-save me-2"></i> 保存账户设置
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- 通知设置 -->
        <div class="card animate__animated animate__fadeIn mb-4">
            <div class="card-header d-flex justify-content-between align-items-center">
                <span>通知设置</span>
                <i class="bi bi-bell"></i>
            </div>
            <div class="card-body">
                <form id="notification-settings-form">
                    <div class="form-check form-switch mb-3">
                        <input class="form-check-input" type="checkbox" id="notify-email" checked>
                        <label class="form-check-label" for="notify-email">电子邮件通知</label>
                    </div>
                    <div class="form-check form-switch mb-3">
                        <input class="form-check-input" type="checkbox" id="notify-sms" checked>
                        <label class="form-check-label" for="notify-sms">短信通知</label>
                    </div>
                    <div class="form-check form-switch mb-3">
                        <input class="form-check-input" type="checkbox" id="notify-push" checked>
                        <label class="form-check-label" for="notify-push">推送通知</label>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">通知类型</label>
                        <div class="form-check">
                            <input class="form-check-input" type="checkbox" id="notify-conflicts" checked>
                            <label class="form-check-label" for="notify-conflicts">冲突提醒</label>
                        </div>
                        <div class="form-check">
                            <input class="form-check-input" type="checkbox" id="notify-schedule" checked>
                            <label class="form-check-label" for="notify-schedule">排课完成</label>
                        </div>
                        <div class="form-check">
                            <input class="form-check-input" type="checkbox" id="notify-system">
                            <label class="form-check-label" for="notify-system">系统更新</label>
                        </div>
                    </div>
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-save me-2"></i> 保存通知设置
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- 个性化设置 -->
        <div class="card animate__animated animate__fadeIn mb-4">
            <div class="card-header d-flex justify-content-between align-items-center">
                <span>个性化设置</span>
                <i class="bi bi-palette"></i>
            </div>
            <div class="card-body">
                <form id="personalization-form">
                    <div class="mb-3">
                        <label for="theme-select" class="form-label">主题颜色</label>
                        <select class="form-select" id="theme-select">
                            <option value="purple">紫色 (默认)</option>
                            <option value="blue">蓝色</option>
                            <option value="green">绿色</option>
                            <option value="red">红色</option>
                            <option value="orange">橙色</option>
                            <option value="dark">深色</option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label for="font-size" class="form-label">字体大小</label>
                        <input type="range" class="form-range" id="font-size" min="12" max="18" step="1" value="14">
                        <div class="d-flex justify-content-between">
                            <small>小</small>
                            <small>中</small>
                            <small>大</small>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">界面密度</label>
                        <div class="btn-group w-100" role="group">
                            <input type="radio" class="btn-check" name="density" id="density-compact" autocomplete="off">
                            <label class="btn btn-outline-primary" for="density-compact">紧凑</label>
                            <input type="radio" class="btn-check" name="density" id="density-normal" autocomplete="off" checked>
                            <label class="btn btn-outline-primary" for="density-normal">正常</label>
                            <input type="radio" class="btn-check" name="density" id="density-spacious" autocomplete="off">
                            <label class="btn btn-outline-primary" for="density-spacious">宽松</label>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">默认视图</label>
                        <select class="form-select" id="default-view">
                            <option value="dashboard">控制面板</option>
                            <option value="scheduling">课程排班</option>
                            <option value="dynamic">动态调度</option>
                            <option value="preferences">偏好匹配</option>
                            <option value="conflicts">冲突检测</option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">动画效果</label>
                        <div class="form-check form-switch">
                            <input class="form-check-input" type="checkbox" id="enable-animations" checked>
                            <label class="form-check-label" for="enable-animations">启用动画效果</label>
                        </div>
                    </div>
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-save me-2"></i> 保存个性化设置
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- 系统配置 -->
        <div class="card animate__animated animate__fadeIn mb-4">
            <div class="card-header d-flex justify-content-between align-items-center">
                <span>系统配置</span>
                <i class="bi bi-gear-wide-connected"></i>
            </div>
            <div class="card-body">
                <form id="system-config-form">
                    <div class="mb-3">
                        <label for="backup-frequency" class="form-label">数据备份频率</label>
                        <select class="form-select" id="backup-frequency">
                            <option value="daily">每天</option>
                            <option value="weekly" selected>每周</option>
                            <option value="monthly">每月</option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label for="auto-update" class="form-label">自动更新</label>
                        <select class="form-select" id="auto-update">
                            <option value="enabled">启用</option>
                            <option value="disabled" selected>禁用</option>
                            <option value="notify">仅通知</option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label for="session-timeout" class="form-label">会话超时(分钟)</label>
                        <input type="number" class="form-control" id="session-timeout" value="30" min="5" max="240">
                    </div>
                    <div class="mb-3">
                        <label for="default-algorithm" class="form-label">默认排课算法</label>
                        <select class="form-select" id="default-algorithm">
                            <option value="genetic">遗传算法</option>
                            <option value="constraint">约束满足</option>
                            <option value="hybrid" selected>混合算法</option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">系统日志</label>
                        <div class="form-check form-switch">
                            <input class="form-check-input" type="checkbox" id="enable-logging" checked>
                            <label class="form-check-label" for="enable-logging">启用详细日志</label>
                        </div>
                    </div>
                    <div class="d-flex justify-content-between mt-4">
                        <button type="button" class="btn btn-outline-secondary" id="export-config">
                            <i class="bi bi-download me-2"></i> 导出配置
                        </button>
                        <button type="button" class="btn btn-outline-primary" id="import-config">
                            <i class="bi bi-upload me-2"></i> 导入配置
                        </button>
                    </div>
                    <div class="d-grid gap-2 mt-3">
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-save me-2"></i> 保存系统配置
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- 高级设置 -->
        <div class="card animate__animated animate__fadeIn">
            <div class="card-header d-flex justify-content-between align-items-center">
                <span>高级设置</span>
                <i class="bi bi-shield-lock"></i>
            </div>
            <div class="card-body">
                <div class="alert alert-warning">
                    <i class="bi bi-exclamation-triangle-fill me-2"></i>
                    警告：修改这些设置可能会影响系统稳定性，请谨慎操作。
                </div>

                <form id="advanced-settings-form">
                    <div class="row">
                        <div class="col-md-4">
                            <div class="mb-3">
                                <label for="max-concurrent-tasks" class="form-label">最大并发任务数</label>
                                <input type="number" class="form-control" id="max-concurrent-tasks" value="5" min="1" max="10">
                            </div>
                        </div>
                        <div class="col-md-4">
                            <div class="mb-3">
                                <label for="db-connection-limit" class="form-label">数据库连接限制</label>
                                <input type="number" class="form-control" id="db-connection-limit" value="20" min="5" max="50">
                            </div>
                        </div>
                        <div class="col-md-4">
                            <div class="mb-3">
                                <label for="cache-size" class="form-label">缓存大小(MB)</label>
                                <input type="number" class="form-control" id="cache-size" value="256" min="64" max="1024">
                            </div>
                        </div>
                    </div>

                    <div class="mb-3">
                        <label for="api-timeout" class="form-label">API超时(秒)</label>
                        <input type="number" class="form-control" id="api-timeout" value="30" min="5" max="120">
                    </div>

                    <div class="mb-3">
                        <label for="log-retention" class="form-label">日志保留天数</label>
                        <input type="number" class="form-control" id="log-retention" value="30" min="7" max="365">
                    </div>

                    <div class="form-check form-switch mb-3">
                        <input class="form-check-input" type="checkbox" id="enable-debug-mode">
                        <label class="form-check-label" for="enable-debug-mode">启用调试模式</label>
                    </div>

                    <div class="form-check form-switch mb-3">
                        <input class="form-check-input" type="checkbox" id="enable-maintenance-mode">
                        <label class="form-check-label" for="enable-maintenance-mode">启用维护模式</label>
                    </div>

                    <div class="d-flex justify-content-end gap-2 mt-4">
                        <button type="button" class="btn btn-outline-danger" id="reset-defaults">
                            <i class="bi bi-arrow-counterclockwise me-2"></i> 恢复默认设置
                        </button>
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-save me-2"></i> 保存高级设置
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
    <!-- WebSocket 状态提示 -->
    <div class="position-fixed bottom-0 end-0 p-3" style="z-index: 11">
        <div id="ws-status" class="toast align-items-center text-white bg-success" role="alert" aria-live="assertive" aria-atomic="true">
            <div class="d-flex">
                <div class="toast-body">
                    <i class="bi bi-check-circle-fill me-2"></i>
                    WebSocket 连接已建立
                </div>
                <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
            </div>
        </div>
    </div>

    <!-- 模态框 - 课程详情 -->
    <div class="modal fade" id="classDetailModal" tabindex="-1" aria-labelledby="classDetailModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="classDetailModalLabel">课程详情</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <div class="row mb-3">
                        <div class="col-md-6">
                            <strong>课程名称:</strong> <span id="modal-class-name">数据结构</span>
                        </div>
                        <div class="col-md-6">
                            <strong>教师:</strong> <span id="modal-class-teacher">张教授</span>
                        </div>
                    </div>
                    <div class="row mb-3">
                        <div class="col-md-6">
                            <strong>教室:</strong> <span id="modal-class-room">A201</span>
                        </div>
                        <div class="col-md-6">
                            <strong>时间:</strong> <span id="modal-class-time">周一 8:00-9:30</span>
                        </div>
                    </div>
                    <div class="mb-3">
                        <strong>学生人数:</strong> <span id="modal-class-students">45</span>
                    </div>
                    <div class="mb-3">
                        <strong>偏好匹配度:</strong>
                        <div class="progress mt-2">
                            <div class="progress-bar bg-success" role="progressbar" style="width: 85%;" aria-valuenow="85" aria-valuemin="0" aria-valuemax="100">85%</div>
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">关闭</button>
                    <button type="button" class="btn btn-primary">编辑课程</button>
                </div>
            </div>
        </div>
    </div>

    <!-- 模态框 - 教室状态编辑 -->
    <div class="modal fade" id="roomStatusModal" tabindex="-1" aria-labelledby="roomStatusModalLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="roomStatusModalLabel">编辑教室状态</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form id="room-status-form">
                        <input type="hidden" id="room-id">
                        <div class="mb-3">
                            <label for="room-status" class="form-label">状态</label>
                            <select class="form-select" id="room-status">
                                <option value="available">可用</option>
                                <option value="occupied">占用中</option>
                                <option value="maintenance">维护中</option>
                            </select>
                        </div>
                        <div class="mb-3">
                            <label for="room-class" class="form-label">当前课程</label>
                            <input type="text" class="form-control" id="room-class">
                        </div>
                        <div class="mb-3">
                            <label for="room-equipment" class="form-label">设备状态</label>
                            <select class="form-select" id="room-equipment">
                                <option value="normal">正常</option>
                                <option value="projector-failure">投影仪故障</option>
                                <option value="ac-failure">空调故障</option>
                                <option value="sound-failure">音响故障</option>
                                <option value="maintenance">维护中</option>
                            </select>
                        </div>
                        <div class="mb-3">
                            <label for="room-notes" class="form-label">备注</label>
                            <textarea class="form-control" id="room-notes" rows="3"></textarea>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">取消</button>
                    <button type="button" class="btn btn-primary" id="save-room-status">保存更改</button>
                </div>
            </div>
        </div>
    </div>
</div>的JavaScript库 -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/apexcharts@3.35.0/dist/apexcharts.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sortablejs@1.14.0/Sortable.min.js"></script>

    <script>
        // 使用Web Workers处理耗时任务
        const schedulerWorker = new Worker('scheduler-worker.js');

        // 教室状态数据
        let roomsData = [
            { id: 'A201', status: 'available', class: '数据结构', equipment: 'normal', notes: '' },
            { id: 'B305', status: 'occupied', class: '算法分析', equipment: 'normal', notes: '' },
            { id: 'C101', status: 'occupied', class: '数据库系统', equipment: 'ac-failure', notes: '空调需要维修' },
            { id: 'D402', status: 'maintenance', class: '-', equipment: 'maintenance', notes: '全面维护中' },
            { id: 'E203', status: 'available', class: '-', equipment: 'normal', notes: '' },
            { id: 'F301', status: 'available', class: '-', equipment: 'normal', notes: '' },
            { id: 'G102', status: 'occupied', class: '人工智能', equipment: 'normal', notes: '' },
            { id: 'H205', status: 'occupied', class: '计算机图形学', equipment: 'projector-failure', notes: '投影仪需要更换灯泡' }
        ];

        // 教室状态变更历史
        let roomStatusHistory = [
            { roomId: 'A201', action: '状态更新', from: 'occupied', to: 'available', timestamp: '2023-06-18 10:30', user: 'admin' },
            { roomId: 'B305', action: '课程安排', class: '算法分析', timestamp: '2023-06-18 09:15', user: 'admin' },
            { roomId: 'C101', action: '设备报修', equipment: '空调故障', timestamp: '2023-06-17 14:20', user: 'teacher1' },
            { roomId: 'D402', action: '维护开始', notes: '全面维护中', timestamp: '2023-06-16 08:00', user: 'maintenance' }
        ];

        // 初始化WebSocket连接
        document.addEventListener('DOMContentLoaded', function() {
            // 模拟WebSocket连接
            setTimeout(() => {
                const wsToast = new bootstrap.Toast(document.getElementById('ws-status'));
                wsToast.show();

                // 模拟实时更新
                setInterval(() => {
                    document.querySelector('.toast-body').innerHTML = `
                        <i class="bi bi-check-circle-fill me-2"></i>
                        WebSocket 连接正常 | 最后更新: ${new Date().toLocaleTimeString()}
                    `;
                }, 30000);
            }, 1000);

            // 初始化图表
            initCharts();

            // 初始化课程表
            initTimetable();

            // 初始化教室状态表
            initRoomStatus();

            // 初始化冲突检测表
            initConflictResults();

            // 初始化偏好匹配表
            initPreferenceMatch();

            // 绑定事件
            bindEvents();

            // 延迟加载非关键资源
            loadLazyResources();

            // 初始化拖放功能
            initDragAndDrop();

            // 监听系统主题变化
            watchSystemTheme();

            // 初始化Intersection Observer
            initIntersectionObserver();
        });

        function initCharts() {
            // 教室状态图表
            const roomStatusChart = new ApexCharts(document.querySelector("#room-status-chart"), {
                series: [12, 8, 2],
                labels: ['可用', '占用中', '维护中'],
                colors: ['#28a745', '#dc3545', '#ffc107'],
                chart: {
                    type: 'donut',
                    height: '100%',
                    animations: {
                        enabled: true,
                        easing: 'easeinout',
                        speed: 800,
                        animateGradually: {
                            enabled: true,
                            delay: 150
                        },
                        dynamicAnimation: {
                            enabled: true,
                            speed: 350
                        }
                    }
                },
                plotOptions: {
                    pie: {
                        donut: {
                            labels: {
                                show: true,
                                total: {
                                    show: true,
                                    label: '教室总数',
                                    color: '#6c757d'
                                }
                            }
                        }
                    }
                },
                dataLabels: {
                    enabled: false
                },
                legend: {
                    position: 'bottom'
                },
                responsive: [{
                    breakpoint: 480,
                    options: {
                        chart: {
                            width: 200
                        },
                        legend: {
                            position: 'bottom'
                        }
                    }
                }]
            });
            roomStatusChart.render();

            // 资源使用率图表
            const resourceUsageChart = new ApexCharts(document.querySelector("#resource-usage-chart"), {
                series: [{
                    name: '使用率',
                    data: [30, 40, 35, 50, 49, 60, 70, 91]
                }],
                chart: {
                    type: 'area',
                    height: '100%',
                    animations: {
                        enabled: true,
                        easing: 'easeinout',
                        speed: 800
                    },
                    toolbar: {
                        show: false
                    }
                },
                colors: ['#6f42c1'],
                dataLabels: {
                    enabled: false
                },
                stroke: {
                    curve: 'smooth',
                    width: 2
                },
                fill: {
                    type: 'gradient',
                    gradient: {
                        shadeIntensity: 1,
                        opacityFrom: 0.7,
                        opacityTo: 0.3,
                        stops: [0, 90, 100]
                    }
                },
                xaxis: {
                    categories: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
                },
                tooltip: {
                    enabled: true,
                    followCursor: true
                }
            });
            resourceUsageChart.render();

            // 算法性能图表
            const algorithmPerformanceChart = new ApexCharts(document.querySelector("#algorithm-performance-chart"), {
                series: [
                    {
                        name: '遗传算法',
                        data: [45, 52, 38, 24, 33, 26, 21, 20, 6, 8, 15, 10]
                    },
                    {
                        name: '约束满足',
                        data: [35, 41, 62, 42, 13, 18, 29, 37, 36, 51, 32, 35]
                    },
                    {
                        name: '混合算法',
                        data: [87, 57, 74, 99, 75, 38, 62, 47, 82, 56, 45, 47]
                    }
                ],
                chart: {
                    type: 'line',
                    height: '100%',
                    animations: {
                        enabled: true,
                        easing: 'linear',
                        dynamicAnimation: {
                            speed: 1000
                        }
                    },
                    toolbar: {
                        show: false
                    },
                    zoom: {
                        enabled: false
                    }
                },
                dataLabels: {
                    enabled: false
                },
                stroke: {
                    width: 3,
                    curve: 'smooth'
                },
                colors: ['#6f42c1', '#6610f2', '#4e0dba'],
                xaxis: {
                    categories: ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '8月', '9月', '10月', '11月', '12月'],
                },
                markers: {
                    size: 5,
                    hover: {
                        size: 7
                    }
                },
                tooltip: {
                    intersect: true,
                    shared: false
                },
                legend: {
                    position: 'top'
                }
            });
            algorithmPerformanceChart.render();
        }

        function initTimetable() {
            const timetableBody = document.querySelector('#timetable tbody');
            const timeSlots = [
                '8:00-9:30', '9:40-11:10', '11:20-12:50',
                '13:30-15:00', '15:10-16:40', '16:50-18:20',
                '18:30-20:00', '20:10-21:40'
            ];

            timetableBody.innerHTML = '';

            timeSlots.forEach(slot => {
                const row = document.createElement('tr');
                row.innerHTML = `<td>${slot}</td>
                                <td class="time-slot"></td>
                                <td class="time-slot"></td>
                                <td class="time-slot"></td>
                                <td class="time-slot"></td>
                                <td class="time-slot"></td>`;
                timetableBody.appendChild(row);
            });

            // 模拟添加课程
            addClassToTimetable('周一', 0, '数据结构', '张教授', 'A201');
            addClassToTimetable('周一', 1, '算法分析', '李教授', 'B305');
            addClassToTimetable('周三', 2, '数据库系统', '王教授', 'A201');
            addClassToTimetable('周三', 3, '机器学习', '赵教授', 'C101');
            addClassToTimetable('周五', 4, '计算机网络', '刘教授', 'B305');

            // 添加一个冲突课程用于演示
            addClassToTimetable('周一', 0, '操作系统', '陈教授', 'A201', true);
        }

        function addClassToTimetable(day, timeIndex, className, teacher, room, isConflict = false) {
            const days = ['周一', '周二', '周三', '周四', '周五'];
            const dayIndex = days.indexOf(day) + 1;
            const rows = document.querySelectorAll('#timetable tbody tr');
            const cell = rows[timeIndex].cells[dayIndex];

            const classElement = document.createElement('div');
            classElement.className = `scheduled-class ${isConflict ? 'conflict' : ''}`;
            classElement.innerHTML = `
                <div class="d-flex justify-content-between align-items-center">
                    <strong>${className}</strong>
                    <i class="bi bi-info-circle ms-2"></i>
                </div>
                <div>${teacher}</div>
                <div>${room}</div>
            `;
            classElement.setAttribute('data-class', className);
            classElement.setAttribute('data-teacher', teacher);
            classElement.setAttribute('data-room', room);

            // 添加点击事件显示模态框
            classElement.addEventListener('click', function() {
                const modal = new bootstrap.Modal(document.getElementById('classDetailModal'));
                document.getElementById('modal-class-name').textContent = className;
                document.getElementById('modal-class-teacher').textContent = teacher;
                document.getElementById('modal-class-room').textContent = room;
                document.getElementById('modal-class-time').textContent = `${day} ${rows[timeIndex].cells[0].textContent}`;
                document.getElementById('modal-class-students').textContent = Math.floor(Math.random() * 30) + 20;
                modal.show();
            });

            cell.appendChild(classElement);
        }

        function initRoomStatus() {
            const roomStatusBody = document.querySelector('#room-status-table tbody');
            updateRoomStatusTable(roomStatusBody);
        }

        function updateRoomStatusTable(container) {
            container.innerHTML = '';

            roomsData.forEach(room => {
                const row = document.createElement('tr');
                let statusBadge = '';
                let statusClass = '';
                let equipmentBadge = '';

                // 设置状态样式
                if (room.status === 'available') {
                    statusBadge = '<span class="badge bg-success">可用</span>';
                    statusClass = 'table-success';
                } else if (room.status === 'occupied') {
                    statusBadge = '<span class="badge bg-danger">占用中</span>';
                    statusClass = 'table-danger';
                } else {
                    statusBadge = '<span class="badge bg-warning text-dark">维护中</span>';
                    statusClass = 'table-warning';
                }

                // 设置设备状态样式
                if (room.equipment === 'normal') {
                    equipmentBadge = '<span class="badge bg-success">正常</span>';
                } else if (room.equipment === 'projector-failure') {
                    equipmentBadge = '<span class="badge bg-warning text-dark">投影仪故障</span>';
                } else if (room.equipment === 'ac-failure') {
                    equipmentBadge = '<span class="badge bg-warning text-dark">空调故障</span>';
                } else if (room.equipment === 'sound-failure') {
                    equipmentBadge = '<span class="badge bg-warning text-dark">音响故障</span>';
                } else {
                    equipmentBadge = '<span class="badge bg-secondary">维护中</span>';
                }

                row.className = statusClass;
                row.innerHTML = `
                    <td>${room.id}</td>
                    <td>${statusBadge}</td>
                    <td>${room.class}</td>
                    <td>${equipmentBadge}</td>
                    <td>
                        <button class="btn btn-sm btn-outline-primary view-room-details" data-room-id="${room.id}" data-bs-toggle="tooltip" title="查看详情">
                            <i class="bi bi-info-circle"></i>
                        </button>
                        <button class="btn btn-sm btn-outline-secondary ms-1 edit-room-status" data-room-id="${room.id}" data-bs-toggle="tooltip" title="编辑">
                            <i class="bi bi-pencil"></i>
                        </button>
                    </td>
                `;
                container.appendChild(row);
            });

            // 初始化工具提示
            const tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'));
            tooltipTriggerList.map(function (tooltipTriggerEl) {
                return new bootstrap.Tooltip(tooltipTriggerEl);
            });

            // 更新统计信息
            updateRoomStatistics();
        }

        function updateRoomStatistics() {
            const availableCount = roomsData.filter(r => r.status === 'available').length;
            const occupiedCount = roomsData.filter(r => r.status === 'occupied').length;
            const maintenanceCount = roomsData.filter(r => r.status === 'maintenance').length;

            document.getElementById('available-rooms').textContent = availableCount;
            document.getElementById('occupied-rooms').textContent = occupiedCount;
            document.getElementById('maintenance-rooms').textContent = maintenanceCount;
        }

        function initConflictResults() {
            const conflictBody = document.querySelector('#conflict-result-table tbody');
            const conflicts = [
                { type: '时间冲突', objects: '张教授, 李教授', time: '周三 10:00-12:00', room: 'A201, B305', severity: '高' },
                { type: '教室冲突', objects: '数据结构, 算法分析', time: '周五 14:00-16:00', room: 'B305', severity: '中' },
                { type: '学生冲突', objects: '计算机科学1班', time: '周二 8:00-10:00', room: 'C101', severity: '低' }
            ];

            conflictBody.innerHTML = '';

            conflicts.forEach(conflict => {
                const row = document.createElement('tr');
                let severityBadge = '';
                let rowClass = '';

                if (conflict.severity === '高') {
                    severityBadge = '<span class="badge bg-danger">高</span>';
                    rowClass = 'table-danger';
                } else if (conflict.severity === '中') {
                    severityBadge = '<span class="badge bg-warning text-dark">中</span>';
                    rowClass = 'table-warning';
                } else {
                    severityBadge = '<span class="badge bg-success">低</span>';
                    rowClass = 'table-success';
                }

                row.className = rowClass;
                row.innerHTML = `
                    <td>${conflict.type}</td>
                    <td>${conflict.objects}</td>
                    <td>${conflict.time}</td>
                    <td>${conflict.room}</td>
                    <td>${severityBadge}</td>
                `;
                conflictBody.appendChild(row);
            });
        }

        function initPreferenceMatch() {
            const matchBody = document.querySelector('#preference-match-table tbody');
            const matches = [
                { course: '数据结构', teacher: '张教授', studentGroup: '计算机科学1班', timeMatch: 85, roomMatch: 90, overall: 88, conflicts: '无' },
                { course: '算法分析', teacher: '李教授', studentGroup: '计算机科学2班', timeMatch: 70, roomMatch: 85, overall: 78, conflicts: '时间偏好部分冲突' },
                { course: '数据库系统', teacher: '王教授', studentGroup: '信息管理1班', timeMatch: 95, roomMatch: 80, overall: 87, conflicts: '无' },
                { course: '机器学习', teacher: '赵教授', studentGroup: '软件工程1班', timeMatch: 65, roomMatch: 75, overall: 70, conflicts: '教室偏好不匹配' }
            ];

            matchBody.innerHTML = '';

            matches.forEach(match => {
                const row = document.createElement('tr');

                // 根据匹配度设置行颜色
                if (match.overall >= 85) {
                    row.className = 'table-success';
                } else if (match.overall >= 70) {
                    row.className = 'table-warning';
                } else {
                    row.className = 'table-danger';
                }

                row.innerHTML = `
                    <td>${match.course}</td>
                    <td>${match.teacher}</td>
                    <td>${match.studentGroup}</td>
                    <td>
                        ${match.timeMatch}%
                        <div class="preference-match">
                            <div class="preference-match-bar" style="width: ${match.timeMatch}%"></div>
                        </div>
                    </td>
                    <td>
                        ${match.roomMatch}%
                        <div class="preference-match">
                            <div class="preference-match-bar" style="width: ${match.roomMatch}%"></div>
                        </div>
                    </td>
                    <td>
                        ${match.overall}%
                        <div class="preference-match">
                            <div class="preference-match-bar" style="width: ${match.overall}%"></div>
                        </div>
                    </td>
                    <td>${match.conflicts}</td>
                `;
                matchBody.appendChild(row);
            });
        }

        function loadLazyResources() {
            // 模拟延迟加载非关键资源
            setTimeout(() => {
                const lazyElements = document.querySelectorAll('.lazy-load');
                lazyElements.forEach(el => {
                    el.classList.add('loaded');
                });
            }, 500);
        }

        function initDragAndDrop() {
            // 初始化拖放功能
            const timeSlots = document.querySelectorAll('.time-slot');
            timeSlots.forEach(slot => {
                new Sortable(slot, {
                    group: 'classes',
                    animation: 150,
                    ghostClass: 'dragging',
                    onEnd: function(evt) {
                        console.log('课程已移动', evt);
                        // 这里可以添加AJAX请求保存新的课程安排
                    }
                });
            });
        }

        function watchSystemTheme() {
            // 监听系统主题变化
            const darkModeMediaQuery = window.matchMedia('(prefers-color-scheme: dark)');

            const handleThemeChange = (e) => {
                if (e.matches) {
                    document.documentElement.setAttribute('data-theme', 'dark');
                } else {
                    document.documentElement.setAttribute('data-theme', 'light');
                }
            };

            darkModeMediaQuery.addListener(handleThemeChange);
            handleThemeChange(darkModeMediaQuery);
        }

        function initIntersectionObserver() {
            // 使用Intersection Observer实现懒加载和动画触发
            const observer = new IntersectionObserver((entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        entry.target.classList.add('animate__animated', 'animate__fadeInUp');
                        observer.unobserve(entry.target);
                    }
                });
            }, { threshold: 0.1 });

            document.querySelectorAll('.card, .table-responsive').forEach(el => {
                observer.observe(el);
            });
        }

        function bindEvents() {
            // 侧边栏切换
            document.getElementById('sidebarToggle').addEventListener('click', function() {
                document.getElementById('sidebar').classList.toggle('active');
            });

            document.getElementById('collapseSidebar').addEventListener('click', function() {
                document.getElementById('sidebar').classList.toggle('sidebar-collapsed');
                document.getElementById('mainContent').classList.toggle('main-content-expanded');

                const icon = this.querySelector('i');
                if (document.getElementById('sidebar').classList.contains('sidebar-collapsed')) {
                    icon.classList.remove('bi-chevron-double-left');
                    icon.classList.add('bi-chevron-double-right');
                } else {
                    icon.classList.remove('bi-chevron-double-right');
                    icon.classList.add('bi-chevron-double-left');
                }

                // 隐藏导航文本
                document.querySelectorAll('.nav-text').forEach(el => {
                    el.classList.toggle('d-none');
                });
            });

            // 主题切换
            document.getElementById('themeSwitcher').addEventListener('click', function() {
                const html = document.documentElement;
                const themeIcon = this.querySelector('i');

                if (html.getAttribute('data-theme') === 'dark') {
                    html.setAttribute('data-theme', 'light');
                    themeIcon.classList.remove('bi-sun');
                    themeIcon.classList.add('bi-moon-stars-fill');
                } else {
                    html.setAttribute('data-theme', 'dark');
                    themeIcon.classList.remove('bi-moon-stars-fill');
                    themeIcon.classList.add('bi-sun');
                }
            });

            // 开始排课按钮
            document.getElementById('start-scheduling').addEventListener('click', function(e) {
                e.preventDefault();
                document.getElementById('scheduling-status').style.display = 'none';
                document.getElementById('task-progress').style.display = 'block';
                document.getElementById('task-id').textContent = 'TASK-' + Math.floor(Math.random() * 10000);

                // 显示加载指示器
                const spinner = document.getElementById('schedulingSpinner');
                spinner.classList.remove('d-none');
                this.setAttribute('disabled', 'disabled');

                // 模拟进度更新
                let progress = 0;
                const progressInterval = setInterval(() => {
                    progress += Math.random() * 10;
                    if (progress > 100) progress = 100;

                    document.getElementById('progress-bar').style.width = progress + '%';
                    document.getElementById('progress-percent').textContent = Math.floor(progress) + '%';

                    // 更新性能指标
                    document.getElementById('cpu-usage').textContent = Math.floor(Math.random() * 30) + 70 + '%';
                    document.getElementById('memory-usage').textContent = Math.floor(Math.random() * 500) + 500 + ' MB';

                    if (progress >= 100) {
                        clearInterval(progressInterval);
                        spinner.classList.add('d-none');

                        // 显示完成状态
                        setTimeout(() => {
                            const toast = new bootstrap.Toast(document.createElement('div'));
                            toast._element.classList.add('toast', 'bg-success', 'text-white');
                            toast._element.innerHTML = `
                                <div class="d-flex">
                                    <div class="toast-body">
                                        <i class="bi bi-check-circle-fill me-2"></i>
                                        排课任务已完成！
                                    </div>
                                    <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                                </div>
                            `;
                            document.body.appendChild(toast._element);
                            toast.show();

                            // 隐藏进度条，显示结果
                            setTimeout(() => {
                                document.getElementById('task-progress').style.display = 'none';
                                document.getElementById('scheduling-status').style.display = 'block';
                                document.getElementById('scheduling-status').innerHTML = `
                                    <div class="alert alert-success">
                                        <i class="bi bi-check-circle-fill me-2"></i>
                                        排课任务已完成！<br>
                                        <small class="text-muted">任务ID: ${document.getElementById('task-id').textContent}</small>
                                    </div>
                                    <button id="view-schedule" class="btn btn-primary me-2">
                                        <i class="bi bi-calendar-week me-2"></i>查看课表
                                    </button>
                                    <button id="export-schedule" class="btn btn-outline-primary">
                                        <i class="bi bi-download me-2"></i>导出课表
                                    </button>
                                `;

                                // 绑定新按钮事件
                                document.getElementById('view-schedule').addEventListener('click', function() {
                                    const tab = new bootstrap.Tab(document.querySelector('a[href="#dashboard"]'));
                                    tab.show();

                                    // 滚动到课表
                                    setTimeout(() => {
                                        document.querySelector('#timetable').scrollIntoView({ behavior: 'smooth' });
                                    }, 300);
                                });

                                document.getElementById('export-schedule').addEventListener('click', function() {
                                    alert('课表导出功能将在完整版中提供');
                                });
                            }, 1000);
                        }, 500);
                    }
                }, 500);
            });

            // 变异率滑块与输入框同步
            document.getElementById('mutation-rate-slider').addEventListener('input', function() {
                document.getElementById('mutation-rate').value = this.value;
            });

            document.getElementById('mutation-rate').addEventListener('input', function() {
                document.getElementById('mutation-rate-slider').value = this.value;
            });

            // 修复策略选择
            document.getElementById('fix-strategy').addEventListener('change', function() {
                const strategy = this.value;
                document.getElementById('manual-fix-options').style.display = 'none';
                document.getElementById('suggestions-container').style.display = 'none';

                if (strategy === 'manual') {
                    document.getElementById('manual-fix-options').style.display = 'block';
                } else if (strategy === 'suggest') {
                    document.getElementById('suggestions-container').style.display = 'block';

                    // 模拟生成建议
                    const suggestions = [
                        '将"数据结构"调整到周三14:00，教室A201',
                        '将"算法分析"调整到周五10:00，教室B305',
                        '交换"数据结构"和"数据库系统"的时间和教室'
                    ];

                    const suggestionsContainer = document.getElementById('fix-suggestions');
                    suggestionsContainer.innerHTML = '';

                    suggestions.forEach((suggestion, index) => {
                        const suggestionElement = document.createElement('button');
                        suggestionElement.type = 'button';
                        suggestionElement.className = 'list-group-item list-group-item-action d-flex justify-content-between align-items-center';
                        suggestionElement.innerHTML = `
                            <span>${suggestion}</span>
                            <span class="badge bg-primary rounded-pill">方案 ${index + 1}</span>
                        `;
                        suggestionElement.addEventListener('click', function() {
                            const toast = new bootstrap.Toast(document.createElement('div'));
                            toast._element.classList.add('toast', 'bg-success', 'text-white');
                            toast._element.innerHTML = `
                                <div class="d-flex">
                                    <div class="toast-body">
                                        <i class="bi bi-check-circle-fill me-2"></i>
                                        已选择建议方案: ${suggestion}
                                    </div>
                                          <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                                </div>
                            `;
                            document.body.appendChild(toast._element);
                            toast.show();
                        });
                        suggestionsContainer.appendChild(suggestionElement);
                    });
                }
            });

            // 运行冲突检测
            document.getElementById('run-conflict-check').addEventListener('click', function(e) {
                e.preventDefault();

                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    检测中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步检测
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 显示结果
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                冲突检测完成，共发现3个冲突
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();

                    // 更新冲突计数
                    document.getElementById('conflict-count').textContent = '3 个冲突';
                }, 2000);
            });

            // 运行偏好匹配
            document.getElementById('run-preference-match').addEventListener('click', function(e) {
                e.preventDefault();

                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    分析中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步分析
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 显示结果
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                偏好匹配分析完成，平均匹配度82%
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();
                }, 2000);
            });

            // 保存教师偏好
            document.getElementById('save-teacher-pref').addEventListener('click', function(e) {
                e.preventDefault();

                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    保存中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步保存
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 显示成功消息
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                教师偏好已保存
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();
                }, 1000);
            });

            // 保存学生偏好
            document.getElementById('save-student-pref').addEventListener('click', function(e) {
                e.preventDefault();

                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    保存中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步保存
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 显示成功消息
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                学生偏好已保存
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();
                }, 1000);
            });

            // 提交动态调整
            document.getElementById('submit-adjustment').addEventListener('click', function(e) {
                e.preventDefault();

                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    提交中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步提交
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 显示成功消息
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                动态调整请求已提交
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();
                }, 1000);
            });

            // 刷新教室状态
            document.getElementById('refresh-status').addEventListener('click', function() {
                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    刷新中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步刷新
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 随机更新教室状态
                    const availableRooms = Math.floor(Math.random() * 5) + 10;
                    const occupiedRooms = Math.floor(Math.random() * 5) + 5;
                    const maintenanceRooms = Math.floor(Math.random() * 3) + 1;

                    document.getElementById('available-rooms').textContent = availableRooms;
                    document.getElementById('occupied-rooms').textContent = occupiedRooms;
                    document.getElementById('maintenance-rooms').textContent = maintenanceRooms;

                    // 显示成功消息
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                教室状态已刷新
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();
                }, 1000);
            });

            // 应用修复
            document.getElementById('apply-fix').addEventListener('click', function() {
                // 显示加载状态
                const originalText = this.innerHTML;
                this.innerHTML = `
                    <span class="spinner-border spinner-border-sm me-2"></span>
                    应用中...
                `;
                this.setAttribute('disabled', 'disabled');

                // 模拟异步应用修复
                setTimeout(() => {
                    this.innerHTML = originalText;
                    this.removeAttribute('disabled');

                    // 显示成功消息
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                冲突修复已应用
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();
                }, 1000);
            });

            // 教室状态筛选
            document.getElementById('filter-available').addEventListener('click', function() {
                const rows = document.querySelectorAll('#room-status-table tbody tr');
                rows.forEach(row => {
                    if (row.classList.contains('table-success')) {
                        row.style.display = '';
                    } else {
                        row.style.display = 'none';
                    }
                });

                this.classList.add('active');
                document.getElementById('filter-all').classList.remove('active');
            });

            document.getElementById('filter-all').addEventListener('click', function() {
                const rows = document.querySelectorAll('#room-status-table tbody tr');
                rows.forEach(row => {
                    row.style.display = '';
                });

                this.classList.add('active');
                document.getElementById('filter-available').classList.remove('active');
            });

            // 页面切换动画
            const tabLinks = document.querySelectorAll('[data-bs-toggle="tab"]');
            tabLinks.forEach(link => {
                link.addEventListener('click', function() {
                    const target = document.querySelector(this.getAttribute('href'));
                    target.classList.remove('animate__fadeIn');
                    setTimeout(() => {
                        target.classList.add('animate__fadeIn');
                    }, 10);
                });
            });

            // 教室状态编辑按钮事件
            document.addEventListener('click', function(e) {
                if (e.target.closest('.edit-room-status')) {
                    const button = e.target.closest('.edit-room-status');
                    const roomId = button.getAttribute('data-room-id');
                    const room = roomsData.find(r => r.id === roomId);

                    if (room) {
                        const modal = new bootstrap.Modal(document.getElementById('roomStatusModal'));

                        // 填充表单数据
                        document.getElementById('room-id').value = room.id;
                        document.getElementById('room-status').value = room.status;
                        document.getElementById('room-class').value = room.class;
                        document.getElementById('room-equipment').value = room.equipment;
                        document.getElementById('room-notes').value = room.notes;

                        modal.show();
                    }
                }

                if (e.target.closest('.view-room-details')) {
                    const button = e.target.closest('.view-room-details');
                    const roomId = button.getAttribute('data-room-id');
                    const room = roomsData.find(r => r.id === roomId);

                    if (room) {
                        // 显示教室详情
                        const toast = new bootstrap.Toast(document.createElement('div'));
                        toast._element.classList.add('toast', 'bg-info', 'text-white');

                        let statusText = '';
                        if (room.status === 'available') {
                            statusText = '<span class="badge bg-success">可用</span>';
                        } else if (room.status === 'occupied') {
                            statusText = '<span class="badge bg-danger">占用中</span>';
                        } else {
                            statusText = '<span class="badge bg-warning text-dark">维护中</span>';
                        }

                        let equipmentText = '';
                        if (room.equipment === 'normal') {
                            equipmentText = '<span class="badge bg-success">正常</span>';
                        } else if (room.equipment === 'projector-failure') {
                            equipmentText = '<span class="badge bg-warning text-dark">投影仪故障</span>';
                        } else if (room.equipment === 'ac-failure') {
                            equipmentText = '<span class="badge bg-warning text-dark">空调故障</span>';
                        } else if (room.equipment === 'sound-failure') {
                            equipmentText = '<span class="badge bg-warning text-dark">音响故障</span>';
                        } else {
                            equipmentText = '<span class="badge bg-secondary">维护中</span>';
                        }

                        toast._element.innerHTML = `
                            <div class="d-flex">
                                <div class="toast-body">
                                    <h6>教室 ${room.id} 详情</h6>
                                    <div class="mb-2"><strong>状态:</strong> ${statusText}</div>
                                    <div class="mb-2"><strong>当前课程:</strong> ${room.class}</div>
                                    <div class="mb-2"><strong>设备状态:</strong> ${equipmentText}</div>
                                    ${room.notes ? `<div><strong>备注:</strong> ${room.notes}</div>` : ''}
                                </div>
                                <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                            </div>
                        `;
                        document.body.appendChild(toast._element);
                        toast.show();
                    }
                }
            });

            // 保存教室状态
            document.getElementById('save-room-status').addEventListener('click', function() {
                const roomId = document.getElementById('room-id').value;
                const roomStatus = document.getElementById('room-status').value;
                const roomClass = document.getElementById('room-class').value;
                const roomEquipment = document.getElementById('room-equipment').value;
                const roomNotes = document.getElementById('room-notes').value;

                // 更新教室数据
                const roomIndex = roomsData.findIndex(r => r.id === roomId);
                if (roomIndex !== -1) {
                    const oldStatus = roomsData[roomIndex].status;

                    // 记录状态变更历史
                    if (oldStatus !== roomStatus) {
                        roomStatusHistory.unshift({
                            roomId: roomId,
                            action: '状态更新',
                            from: oldStatus,
                            to: roomStatus,
                            timestamp: new Date().toLocaleString(),
                            user: 'admin'
                        });
                    }

                    roomsData[roomIndex] = {
                        ...roomsData[roomIndex],
                        status: roomStatus,
                        class: roomClass,
                        equipment: roomEquipment,
                        notes: roomNotes
                    };

                    // 更新表格
                    updateRoomStatusTable(document.querySelector('#room-status-table tbody'));

                    // 显示成功消息
                    const toast = new bootstrap.Toast(document.createElement('div'));
                    toast._element.classList.add('toast', 'bg-success', 'text-white');
                    toast._element.innerHTML = `
                        <div class="d-flex">
                            <div class="toast-body">
                                <i class="bi bi-check-circle-fill me-2"></i>
                                教室状态已更新
                            </div>
                            <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                        </div>
                    `;
                    document.body.appendChild(toast._element);
                    toast.show();

                    // 关闭模态框
                    bootstrap.Modal.getInstance(document.getElementById('roomStatusModal')).hide();
                }
            });
        }
    </script>

    <!-- Web Worker 脚本 (内联) -->
    <script id="scheduler-worker" type="javascript/worker">
        self.onmessage = function(e) {
            const { algorithm, iterations, populationSize, mutationRate } = e.data;

            // 模拟遗传算法运算
            let progress = 0;
            const interval = setInterval(() => {
                progress += Math.random() * 5;
                if (progress >= 100) {
                    progress = 100;
                    clearInterval(interval);
                }

                self.postMessage({
                    progress: progress,
                    fitness: Math.floor(Math.random() * 100),
                    conflicts: Math.floor(Math.random() * 10)
                });
            }, 200);
        };
    </script>

    <script>
        // 将Web Worker脚本转换为Blob URL
        const workerScript = document.getElementById('scheduler-worker').textContent;
        const blob = new Blob([workerScript], { type: 'text/javascript' });
        const workerUrl = URL.createObjectURL(blob);

        // 创建Web Worker
        const schedulerWorker = new Worker(workerUrl);

        // 监听Web Worker消息
        schedulerWorker.onmessage = function(e) {
            const { progress, fitness, conflicts } = e.data;

            document.getElementById('progress-bar').style.width = progress + '%';
            document.getElementById('progress-percent').textContent = Math.floor(progress) + '%';

            if (progress >= 100) {
                // 排课完成
                schedulerWorker.terminate();
            }
        };

        // 开始排课时向Web Worker发送数据
        document.getElementById('scheduling-form').addEventListener('submit', function(e) {
            e.preventDefault();

            const algorithm = document.getElementById('algorithm-type').value;
            const iterations = document.getElementById('max-iterations').value;
            const populationSize = document.getElementById('population-size').value;
            const mutationRate = document.getElementById('mutation-rate').value;

            schedulerWorker.postMessage({
                algorithm,
                iterations,
                populationSize,
                mutationRate
            });
        });
    </script>
</body>
</html>
