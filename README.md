<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>【mo+】Project Gantt Chart</title>
    <style>
        :root {
            --bg-body: #f8f9fa;
            --bg-card: #ffffff;
            --text-main: #2b3a67;
            --text-light: #6c757d;
            --border-color: #e9ecef;
            --radius-lg: 12px;
        }

        body {
            font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif;
            background: var(--bg-body);
            color: var(--text-main);
            margin: 0;
            padding: 30px;
        }

        .header-section {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            margin-bottom: 20px;
            background: var(--bg-card);
            padding: 20px 30px;
            border-radius: var(--radius-lg);
            box-shadow: 0 4px 15px rgba(0,0,0,0.03);
        }

        .header-title h1 { margin: 0; color: var(--text-main); font-size: 28px; font-weight: 800; }
        .header-title p { margin: 5px 0 0; color: var(--text-light); font-size: 14px; }

        .controls { display: flex; gap: 10px; }
        button {
            padding: 8px 16px; border: none; border-radius: 20px; cursor: pointer;
            font-weight: 600; font-size: 13px; transition: 0.2s;
        }
        .btn-primary { background: var(--text-main); color: white; }
        .btn-primary:hover { background: #1a2540; }
        .btn-outline { background: white; border: 1px solid var(--text-main); color: var(--text-main); }
        .btn-outline:hover { background: #f8f9fa; }
        .btn-danger { background: #ff4757; color: white; }
        .btn-success { background: #2ed573; color: white; }
        .btn-sm { padding: 4px 10px; font-size: 11px; border-radius: 12px; margin-left: 5px;}

        /* Gantt Chart Container */
        .gantt-card {
            background: var(--bg-card);
            border-radius: var(--radius-lg);
            box-shadow: 0 4px 20px rgba(0,0,0,0.05);
            border: 1px solid var(--border-color);
            overflow-x: auto;
        }

        .gantt-grid {
            display: grid;
            grid-template-columns: 250px repeat(30, 100px); 
            background: white;
            gap: 0; 
            min-width: max-content;
        }

        .cell {
            background: white;
            padding: 6px 10px;
            font-size: 13px;
            display: flex;
            align-items: center;
            border-bottom: 1px solid #f0f0f0;
            border-right: 1px solid #f0f0f0;
            box-sizing: border-box;
            height: 50px;
        }

        .cell.header { 
            font-weight: 700; color: var(--text-main); justify-content: center; 
            background: #f8f9fa; font-size: 12px; height: 50px;
            position: sticky; top: 0; z-index: 5;
        }
        .cell.header.task-header { 
            justify-content: flex-start; padding-left: 20px; font-size: 16px;
            position: sticky; left: 0; z-index: 25; background: #f8f9fa; 
            border-right: 2px solid var(--border-color);
        }

        .cell.group-row {
            grid-column: 1 / 2;
            background: #f8f9fa;
            font-weight: 700;
            color: #425cc8;
            font-size: 16px;
            padding: 6px 20px;
            position: sticky; left: 0; z-index: 15; 
            border-right: 2px solid var(--border-color);
        }
        .cell.group-row-filler {
            grid-column: 2 / -1;
            background: #f8f9fa;
            border-bottom: 1px solid #e9ecef;
        }

        .cell.task-name {
            font-weight: 500;
            position: sticky; left: 0; z-index: 20;
            background: white;
            border-right: 2px solid var(--border-color);
            padding-left: 20px;
            flex-direction: column;
            align-items: flex-start;
            justify-content: center;
            cursor: pointer;
        }
        .cell.task-name:hover { background: #f8f9fa; }
        .task-name-text { font-size: 13px; margin-bottom: 2px;}
        .dep-tag {
            font-size: 10px; color: #868e96; background: #e9ecef;
            padding: 2px 6px; border-radius: 8px; display: inline-block;
        }
        .dep-tag.warning { color: #d63384; background: #fcc2d7; }

        .cell.timeline-area {
            grid-column: 2 / -1;
            position: relative;
            padding: 0 0 0 10px; /* 新增左側 padding 創造間距 */
            height: 50px;
            background: white;
            border-right: none;
            box-sizing: border-box;
        }
        
        /* The Bar - 高度 16px，垂直居中 (50-16)/2 = 17px */
        .bar {
            position: absolute;
            top: 17px; 
            height: 16px; 
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 11px;
            font-weight: 600;
            cursor: pointer;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            transition: transform 0.1s;
            z-index: 10;
            overflow: hidden;
            white-space: nowrap;
            padding: 0 8px;
        }
        .bar:hover { transform: scaleY(1.2); z-index: 20; filter: brightness(1.1); }
        
        /* Milestone - 垂直居中調整 */
        .milestone {
            position: absolute;
            top: 8px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 2px;
            cursor: pointer;
            z-index: 10;
        }
        .milestone-label {
            font-size: 11px;
            font-weight: 600;
            color: #333;
            white-space: nowrap;
            order: -1;
        }
        .milestone-shape {
            width: 16px;
            height: 16px;
            border-radius: 3px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
            flex-shrink: 0;
            transition: transform 0.1s;
        }
        .milestone:hover .milestone-shape { transform: scale(1.2); }

        .bg-conflict { border: 2px dashed #ff4757 !important; opacity: 0.8; }

        /* Modal */
        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(43, 58, 103, 0.4); z-index: 100;
            justify-content: center; align-items: center; backdrop-filter: blur(2px);
        }
        .modal-overlay.active { display: flex; }
        .modal {
            background: white; padding: 25px; border-radius: 16px; width: 500px; max-width: 90%;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2); max-height: 90vh; overflow-y: auto;
        }
        .modal h3 { margin-top: 0; color: var(--text-main); border-bottom: 2px solid var(--border-color); padding-bottom: 10px;}
        
        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; font-size: 12px; font-weight: 600; color: var(--text-light); margin-bottom: 5px; }
        .form-control {
            width: 100%; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px;
            font-family: inherit; font-size: 13px; box-sizing: border-box;
        }
        .form-row { display: flex; gap: 10px; }
        .form-row .form-group { flex: 1; }
        
        .checkbox-group { display: flex; align-items: center; gap: 8px; font-size: 13px; }
        
        .week-details-box {
            background: #f8f9fa; padding: 10px; border-radius: 8px; margin-top: 10px;
            max-height: 150px; overflow-y: auto; font-size: 12px;
        }
        .week-detail-item { display: flex; gap: 5px; margin-bottom: 5px; }
        .week-detail-item input { flex: 1; padding: 4px; border: 1px solid #ddd; border-radius: 4px; }

        .modal-footer { display: flex; justify-content: space-between; margin-top: 20px; border-top: 1px solid var(--border-color); padding-top: 15px;}
    </style>
</head>
<body>

    <div class="header-section">
        <div class="header-title">
            <h1>【mo+】Project Gantt Chart</h1>
            <p>Tasks, dependencies, and milestones.</p>
        </div>
        <div class="controls">
            <button class="btn-outline" id="addGroupBtn" onclick="addGroup()">+ New Group</button>
            <button class="btn-primary" id="addTaskBtn" onclick="openModal()">+ New Task</button>
            <button class="btn-outline" id="viewToggleBtn" onclick="toggleViewMode()">👁️ View</button>
            <button class="btn-success" id="saveBtn" onclick="saveData(true)">💾 Save</button>
        </div>
    </div>

    <div class="gantt-card">
        <div class="gantt-grid" id="ganttGrid"></div>
    </div>

    <!-- Edit Modal -->
    <div class="modal-overlay" id="modalOverlay">
        <div class="modal">
            <h3 id="modalTitle">Edit Task</h3>
            
            <div class="form-group">
                <label>Task Name</label>
                <input type="text" id="taskName" class="form-control" placeholder="e.g. Local Kick-off">
            </div>

            <div class="form-row">
                <div class="form-group">
                    <label>Group / Category</label>
                    <select id="taskGroup" class="form-control"></select>
                </div>
                <div class="form-group">
                    <label>Duration (Weeks)</label>
                    <div style="display:flex; gap:5px; align-items:center; font-size:12px;">
                        <select id="startWeek" class="form-control"></select>
                        <span>to</span>
                        <select id="endWeek" class="form-control"></select>
                    </div>
                </div>
            </div>

            <div class="form-group">
                <label>Custom Color (Hex)</label>
                <div style="display:flex; gap:10px; align-items:center;">
                    <input type="color" id="taskColorPicker" style="width:40px; height:35px; border:none; cursor:pointer; background:none;">
                    <input type="text" id="taskColorHex" class="form-control" placeholder="#RRGGBB (Leave empty for default group color)">
                </div>
            </div>

            <div class="form-group checkbox-group">
                <input type="checkbox" id="isMilestone">
                <label for="isMilestone" style="margin:0; cursor:pointer;"> Mark as Milestone (Square)</label>
            </div>

            <div class="form-group">
                <label>Dependencies (Predecessors)</label>
                <select id="taskDeps" class="form-control" multiple style="height: 60px;"></select>
                <small style="color:#888;">Hold Ctrl/Cmd to select multiple</small>
            </div>

            <div class="form-group">
                <label>Weekly Details / Notes</label>
                <div id="weekDetailsContainer" class="week-details-box">
                    <p style="color:#999; text-align:center;">Select duration to add details</p>
                </div>
            </div>

            <div class="modal-footer">
                <button class="btn-danger" id="btnDelete" onclick="deleteTask()" style="display:none;">Delete</button>
                <div style="display:flex; gap:10px; margin-left:auto;">
                    <button class="btn-outline" onclick="closeModal()">Cancel</button>
                    <button class="btn-primary" onclick="saveTask()">Save Task</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        const weeksData = [
            { id: "W35", year: 2026 }, { id: "W36", year: 2026 }, { id: "W37", year: 2026 },
            { id: "W38", year: 2026 }, { id: "W39", year: 2026 }, { id: "W40", year: 2026 },
            { id: "W41", year: 2026 }, { id: "W42", year: 2026 }, { id: "W43", year: 2026 },
            { id: "W44", year: 2026 }, { id: "W45", year: 2026 }, { id: "W46", year: 2026 },
            { id: "W47", year: 2026 }, { id: "W48", year: 2026 }, { id: "W49", year: 2026 },
            { id: "W50", year: 2026 }, { id: "W51", year: 2026 }, { id: "W52", year: 2026 },
            { id: "W1", year: 2027 }, { id: "W2", year: 2027 }, { id: "W3", year: 2027 },
            { id: "W4", year: 2027 }, { id: "W5", year: 2027 }, { id: "W6", year: 2027 },
            { id: "W7", year: 2027 }, { id: "W8", year: 2027 }, { id: "W9", year: 2027 },
            { id: "W10", year: 2027 }, { id: "W11", year: 2027 }, { id: "W12", year: 2027 }
        ];

        const defaultData = {
            groups: [
                { id: "g1", name: "Meeting", color: "#FFD166" },
                { id: "g2", name: "On Progress", color: "#EF476F" },
                { id: "g3", name: "On Going", color: "#7209B7" }
            ],
            tasks: [
                { id: "t1", groupId: "g1", name: "Local Kick-off", start: "W35", end: "W35", isMilestone: true, deps: [], color: "", details: { "W35": "8/26 Kick-off" } },
                { id: "t2", groupId: "g1", name: "Stakeholder Identification", start: "W36", end: "W37", isMilestone: false, deps: ["t1"], color: "", details: {} },
                { id: "t3", groupId: "g1", name: "Developing a Project Plan", start: "W36", end: "W39", isMilestone: false, deps: ["t1"], color: "", details: {} },
                
                { id: "t4", groupId: "g2", name: "Assemble the Project Team", start: "W39", end: "W41", isMilestone: false, deps: ["t3"], color: "", details: {} },
                { id: "t5", groupId: "g2", name: "Resource Allocation", start: "W40", end: "W42", isMilestone: false, deps: ["t4"], color: "", details: {} },
                { id: "t6", groupId: "g2", name: "Assign Roles and Tasks", start: "W41", end: "W43", isMilestone: false, deps: ["t5"], color: "", details: {} },
                
                { id: "t7", groupId: "g3", name: "Initial Launch", start: "W44", end: "W44", isMilestone: true, deps: ["t6"], color: "", details: { "W44": "Grand Opening!" } },
                { id: "t8", groupId: "g3", name: "Hold Weekly Progress", start: "W43", end: "W46", isMilestone: false, deps: [], color: "", details: {} },
                { id: "t9", groupId: "g3", name: "Aligning Tasks", start: "W43", end: "W46", isMilestone: false, deps: [], color: "", details: {} }
            ]
        };

        let projectData = JSON.parse(localStorage.getItem('moplus_gantt_v5')) || JSON.parse(JSON.stringify(defaultData));
        let currentTaskId = null;
        let isViewMode = false;

        function getTaskColor(task, group) {
            return (task.color && /^#[0-9A-F]{6}$/i.test(task.color)) ? task.color : group.color;
        }

        function render() {
            const grid = document.getElementById('ganttGrid');
            grid.innerHTML = '';

            grid.innerHTML += `<div class="cell header task-header">Task List</div>`;
            weeksData.forEach((w, index) => {
                let label = w.id;
                if (index === 0 || w.year !== weeksData[index-1].year) {
                    label = w.year + " " + w.id;
                }
                grid.innerHTML += `<div class="cell header">${label}</div>`;
            });

            projectData.groups.forEach(group => {
                const groupActions = isViewMode ? '' : `
                    <button class="btn-sm btn-outline" onclick="editGroup('${group.id}')">Edit</button>
                    <button class="btn-sm btn-danger" onclick="deleteGroup('${group.id}')">Delete</button>
                `;
                grid.innerHTML += `<div class="cell group-row">${group.name} ${groupActions}</div>`;
                grid.innerHTML += `<div class="cell group-row-filler"></div>`;

                const groupTasks = projectData.tasks.filter(t => t.groupId === group.id);
                groupTasks.forEach(task => {
                    let depHtml = '';
                    if (task.deps && task.deps.length > 0) {
                        const depNames = task.deps.map(d => {
                            const depTask = projectData.tasks.find(t => t.id === d);
                            return depTask ? depTask.name : d;
                        }).join(', ');
                        const hasConflict = checkConflict(task);
                        depHtml = `<span class="dep-tag ${hasConflict ? 'warning' : ''}">⬅ ${depNames}</span>`;
                    }
                    
                    const clickAction = isViewMode ? '' : `onclick="openModal('${task.id}')"`;
                    grid.innerHTML += `
                        <div class="cell task-name" ${clickAction}>
                            <span class="task-name-text">${task.name}</span>
                            ${depHtml}
                        </div>
                    `;

                    const startIdx = weeksData.findIndex(w => w.id === task.start);
                    const endIdx = weeksData.findIndex(w => w.id === task.end);
                    const totalWeeks = weeksData.length;
                    
                    if (startIdx === -1 || endIdx === -1) return;

                    const weekWidthPct = 100 / totalWeeks;
                    const bgColor = getTaskColor(task, group);
                    const conflictClass = checkConflict(task) ? 'bg-conflict' : '';

                    let barHtml = '';
                    if (task.isMilestone) {
                        const label = task.details[task.start] || task.name;
                        // 計算中心點並加上 10px 偏移
                        const centerPct = (startIdx * weekWidthPct) + (weekWidthPct / 2);
                        barHtml = `
                            <div class="milestone" style="left: calc(10px + ${centerPct}% - 8px);" ${clickAction}>
                                <div class="milestone-label">${label}</div>
                                <div class="milestone-shape" style="background-color: ${bgColor};"></div>
                            </div>
                        `;
                    } else {
                        const detailsText = Object.values(task.details || {}).join(' | ');
                        const leftPct = startIdx * weekWidthPct;
                        const widthPct = (endIdx - startIdx + 1) * weekWidthPct;
                        // 左邊加 10px 偏移，寬度減 10px 防止超出右邊界
                        barHtml = `
                            <div class="bar ${conflictClass}" 
                                 style="left: calc(10px + ${leftPct}%); width: calc(${widthPct}% - 10px); background-color: ${bgColor};" 
                                 ${clickAction}>
                                ${detailsText}
                            </div>
                        `;
                    }
                    grid.innerHTML += `<div class="cell timeline-area">${barHtml}</div>`;
                });
            });
        }

        function checkConflict(task) {
            if (!task.deps || task.deps.length === 0) return false;
            const taskStartIdx = weeksData.findIndex(w => w.id === task.start);
            for (let depId of task.deps) {
                const depTask = projectData.tasks.find(t => t.id === depId);
                if (depTask && weeksData.findIndex(w => w.id === depTask.end) >= taskStartIdx) return true;
            }
            return false;
        }

        function openModal(taskId = null) {
            if (isViewMode) return;
            currentTaskId = taskId;
            const modal = document.getElementById('modalOverlay');
            const title = document.getElementById('modalTitle');
            const btnDelete = document.getElementById('btnDelete');
            
            const startSelect = document.getElementById('startWeek');
            const endSelect = document.getElementById('endWeek');
            const groupSelect = document.getElementById('taskGroup');
            const depSelect = document.getElementById('taskDeps');
            
            startSelect.innerHTML = ''; endSelect.innerHTML = ''; groupSelect.innerHTML = ''; depSelect.innerHTML = '';
            
            weeksData.forEach(w => {
                const label = `${w.year} ${w.id}`;
                startSelect.innerHTML += `<option value="${w.id}">${label}</option>`;
                endSelect.innerHTML += `<option value="${w.id}">${label}</option>`;
            });

            projectData.groups.forEach(g => {
                groupSelect.innerHTML += `<option value="${g.id}">${g.name}</option>`;
            });

            projectData.tasks.forEach(t => {
                if (t.id !== taskId) depSelect.innerHTML += `<option value="${t.id}">${t.name}</option>`;
            });

            if (taskId) {
                const task = projectData.tasks.find(t => t.id === taskId);
                title.innerText = `Edit: ${task.name}`;
                btnDelete.style.display = 'block';
                
                document.getElementById('taskName').value = task.name;
                document.getElementById('taskGroup').value = task.groupId;
                document.getElementById('startWeek').value = task.start;
                document.getElementById('endWeek').value = task.end;
                document.getElementById('isMilestone').checked = task.isMilestone;
                
                const colorVal = task.color || '';
                document.getElementById('taskColorHex').value = colorVal;
                document.getElementById('taskColorPicker').value = colorVal || '#000000';

                Array.from(depSelect.options).forEach(opt => { opt.selected = task.deps.includes(opt.value); });
                renderWeekDetails(task);
            } else {
                title.innerText = 'New Task';
                btnDelete.style.display = 'none';
                document.getElementById('taskName').value = '';
                document.getElementById('isMilestone').checked = false;
                document.getElementById('taskColorHex').value = '';
                document.getElementById('taskColorPicker').value = '#000000';
                document.getElementById('weekDetailsContainer').innerHTML = '<p style="color:#999; text-align:center;">Save first to add details</p>';
            }
            modal.classList.add('active');
        }

        function renderWeekDetails(task) {
            const container = document.getElementById('weekDetailsContainer');
            container.innerHTML = '';
            const startIdx = weeksData.findIndex(w => w.id === task.start);
            const endIdx = weeksData.findIndex(w => w.id === task.end);
            const relevantWeeks = weeksData.slice(startIdx, endIdx + 1);

            relevantWeeks.forEach(w => {
                const val = task.details && task.details[w.id] ? task.details[w.id] : '';
                container.innerHTML += `
                    <div class="week-detail-item">
                        <span style="font-weight:bold; width:70px;">${w.year} ${w.id}:</span>
                        <input type="text" class="detail-input" data-week="${w.id}" value="${val}" placeholder="Key task/date...">
                    </div>
                `;
            });
        }

        function saveTask() {
            const name = document.getElementById('taskName').value;
            if (!name) return alert('Task Name is required');

            const groupId = document.getElementById('taskGroup').value;
            const start = document.getElementById('startWeek').value;
            const end = document.getElementById('endWeek').value;
            const isMilestone = document.getElementById('isMilestone').checked;
            const color = document.getElementById('taskColorHex').value.trim();
            
            const depSelect = document.getElementById('taskDeps');
            const deps = Array.from(depSelect.selectedOptions).map(o => o.value);

            const details = {};
            document.querySelectorAll('.detail-input').forEach(input => {
                if (input.value.trim()) details[input.dataset.week] = input.value;
            });

            if (weeksData.findIndex(w => w.id === start) > weeksData.findIndex(w => w.id === end)) return alert('Start week must be before End week');

            if (currentTaskId) {
                const task = projectData.tasks.find(t => t.id === currentTaskId);
                Object.assign(task, { name, groupId, start, end, isMilestone, color, deps, details });
            } else {
                projectData.tasks.push({ id: 't' + Date.now(), groupId, name, start, end, isMilestone, color, deps, details });
            }

            closeModal();
            render();
            saveData(false);
        }

        function deleteTask() {
            if (confirm('Delete this task?')) {
                projectData.tasks = projectData.tasks.filter(t => t.id !== currentTaskId);
                projectData.tasks.forEach(t => { t.deps = t.deps.filter(d => d !== currentTaskId); });
                closeModal();
                render();
                saveData(false);
            }
        }

        function addGroup() {
            if (isViewMode) return;
            const name = prompt("Enter new Group name:");
            if (!name) return;
            const id = "g" + Date.now();
            projectData.groups.push({ id: id, name: name, color: "#6c757d" });
            render();
            saveData(false);
        }

        function editGroup(groupId) {
            if (isViewMode) return;
            const group = projectData.groups.find(g => g.id === groupId);
            const newName = prompt("Edit Group name:", group.name);
            if (newName) {
                group.name = newName;
                render();
                saveData(false);
            }
        }

        function deleteGroup(groupId) {
            if (isViewMode) return;
            const hasTasks = projectData.tasks.some(t => t.groupId === groupId);
            if (hasTasks) {
                if (!confirm('This group contains tasks. Delete group and all its tasks?')) return;
                projectData.tasks = projectData.tasks.filter(t => t.groupId !== groupId);
            } else {
                if (!confirm('Delete this group?')) return;
            }
            projectData.groups = projectData.groups.filter(g => g.id !== groupId);
            render();
            saveData(false);
        }

        function toggleViewMode() {
            isViewMode = !isViewMode;
            const btn = document.getElementById('viewToggleBtn');
            if (isViewMode) {
                btn.innerText = '✏️ Edit';
                btn.classList.remove('btn-outline');
                btn.classList.add('btn-primary');
                document.getElementById('addGroupBtn').style.display = 'none';
                document.getElementById('addTaskBtn').style.display = 'none';
                document.getElementById('saveBtn').style.display = 'none';
            } else {
                btn.innerText = '👁️ View';
                btn.classList.remove('btn-primary');
                btn.classList.add('btn-outline');
                document.getElementById('addGroupBtn').style.display = 'block';
                document.getElementById('addTaskBtn').style.display = 'block';
                document.getElementById('saveBtn').style.display = 'block';
            }
            render();
        }

        function closeModal() {
            document.getElementById('modalOverlay').classList.remove('active');
        }

        function saveData(showAlert = false) {
            localStorage.setItem('moplus_gantt_v5', JSON.stringify(projectData));
            if (showAlert) {
                const toast = document.createElement('div');
                toast.innerText = '✅ Saved successfully!';
                toast.style.cssText = 'position:fixed; bottom:20px; right:20px; background:#2ed573; color:white; padding:10px 20px; border-radius:20px; font-weight:bold; box-shadow:0 4px 10px rgba(0,0,0,0.2); z-index:999;';
                document.body.appendChild(toast);
                setTimeout(() => toast.remove(), 2000);
            }
        }

        document.getElementById('taskColorPicker').addEventListener('input', (e) => {
            document.getElementById('taskColorHex').value = e.target.value;
        });
        document.getElementById('taskColorHex').addEventListener('input', (e) => {
            if(/^#[0-9A-F]{6}$/i.test(e.target.value)) {
                document.getElementById('taskColorPicker').value = e.target.value;
            }
        });

        render();
    </script>
</body>
</html>
