<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>轮盘选择器</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 30px; }
    .result { font-size: 1.5em; margin: 20px 0; }
    button { margin: 5px; padding: 8px 16px; }
    .add-form, #editSection { margin-top: 20px; }
    label { margin-right: 8px; }
    .edit-group { margin-bottom: 18px; padding-bottom: 8px; border-bottom: 1px solid #eee; }
    .edit-group label { font-weight: bold; display: block; margin-bottom: 6px; }
    .edit-row { margin-bottom: 6px; }
    .edit-row input { width: 60%; }
    .edit-row button { margin-left: 8px; }
  </style>
</head>
<body>
  <h2>轮盘选择器</h2>
  <label>
    请选择类型：
    <select id="typeSelect" onchange="resetType()">
      <option value="">--请选择--</option>
      <option value="早餐">早餐</option>
      <option value="午餐">午餐</option>
      <option value="晚餐">晚餐</option>
      <option value="宵夜">宵夜</option>
    </select>
  </label>
  <button onclick="randomPick()" id="randomBtn" disabled>随机选择</button>
  <button onclick="changeType()" id="changeBtn" disabled>换一个类型</button>
  <button onclick="toggleEdit()">编辑</button>
  <div class="result" id="result"></div>
  <div class="add-form">
    <h4>新增选项</h4>
    <label>
      类型：
      <select id="addTypeSelect">
        <option value="早餐">早餐</option>
        <option value="午餐">午餐</option>
        <option value="晚餐">晚餐</option>
        <option value="宵夜">宵夜</option>
      </select>
    </label>
    <label>
      新选项：<input type="text" id="newItem" placeholder="输入新选项">
    </label>
    <button onclick="addItem()">新增</button>
  </div>
  <!-- 编辑区，默认隐藏 -->
  <div id="editSection" style="display:none;">
    <h4>编辑所有类型选项</h4>
    <form id="editForm">
      <!-- 动态生成内容 -->
    </form>
    <button onclick="saveEdit()">保存</button>
    <button type="button" onclick="toggleEdit()">关闭</button>
  </div>
  <script>
    const types = {
      '早餐': ['吐司', '面包', '牛奶', '鸡蛋'],
      '午餐': ['便当', '沙拉', '果汁'],
      '晚餐': ['意大利面', '披萨', '红酒'],
      '宵夜': ['泡面', '水果', '牛奶']
    };
    let shownTypes = [];
    let currentType = null;
    let currentItem = null;

    function getRandomType(exclude = []) {
      const available = Object.keys(types).filter(t => !exclude.includes(t));
      if (available.length === 0) return null;
      return available[Math.floor(Math.random() * available.length)];
    }

    function getRandomItem(type) {
      const items = types[type];
      return items[Math.floor(Math.random() * items.length)];
    }

    function resetType() {
      currentType = document.getElementById('typeSelect').value;
      shownTypes = [];
      currentItem = null;
      document.getElementById('result').innerText = '';
      document.getElementById('randomBtn').disabled = !currentType;
      document.getElementById('changeBtn').disabled = true;
    }

    function randomPick() {
      if (!currentType) return;
      shownTypes = [currentType];
      currentItem = getRandomItem(currentType);
      showResult();
      document.getElementById('changeBtn').disabled = false;
    }

    function changeType() {
      if (shownTypes.length === Object.keys(types).length) {
        document.getElementById('result').innerText = '所有类型都已出现过，请重新选择类型。';
        document.getElementById('changeBtn').disabled = true;
        return;
      }
      currentType = getRandomType(shownTypes);
      currentItem = getRandomItem(currentType);
      shownTypes.push(currentType);
      document.getElementById('typeSelect').value = currentType;
      showResult();
    }

    function showResult() {
      document.getElementById('result').innerText =
        `类型：${currentType}\n选项：${currentItem}`;
    }

    function addItem() {
      const type = document.getElementById('addTypeSelect').value;
      const newItem = document.getElementById('newItem').value.trim();
      if (newItem) {
        types[type].push(newItem);
        alert(`已新增到${type}：${newItem}`);
        document.getElementById('newItem').value = '';
        renderEditSection();
      }
    }

    // 编辑区直接显示
    function renderEditSection() {
      const form = document.getElementById('editForm');
      form.innerHTML = '';
      Object.keys(types).forEach(type => {
        const group = document.createElement('div');
        group.className = 'edit-group';
        const label = document.createElement('label');
        label.innerText = type;
        group.appendChild(label);
        types[type].forEach((item, idx) => {
          const row = document.createElement('div');
          row.className = 'edit-row';
          const input = document.createElement('input');
          input.type = 'text';
          input.value = item;
          input.dataset.type = type;
          input.dataset.idx = idx;
          row.appendChild(input);
          const delBtn = document.createElement('button');
          delBtn.type = 'button';
          delBtn.innerText = '删除';
          delBtn.onclick = function() {
            types[type].splice(idx, 1);
            renderEditSection();
          };
          row.appendChild(delBtn);
          group.appendChild(row);
        });
        // 新增空白输入
        const newRow = document.createElement('div');
        newRow.className = 'edit-row';
        const newInput = document.createElement('input');
        newInput.type = 'text';
        newInput.placeholder = '新增选项';
        newInput.dataset.type = type;
        newInput.dataset.idx = 'new';
        newRow.appendChild(newInput);
        group.appendChild(newRow);
        form.appendChild(group);
      });
    }

    function saveEdit() {
      const form = document.getElementById('editForm');
      Object.keys(types).forEach(type => {
        const inputs = form.querySelectorAll(`input[data-type="${type}"]`);
        const newArr = [];
        inputs.forEach(input => {
          const val = input.value.trim();
          if (input.dataset.idx !== 'new' && val) {
            newArr.push(val);
          }
        });
        // 检查新增
        inputs.forEach(input => {
          if (input.dataset.idx === 'new') {
            const val = input.value.trim();
            if (val) newArr.push(val);
          }
        });
        types[type] = newArr;
      });
      renderEditSection();
      alert('已保存所有类型选项');
    }

    function toggleEdit() {
      const editDiv = document.getElementById('editSection');
      if (editDiv.style.display === 'none') {
        renderEditSection();
        editDiv.style.display = 'block';
      } else {
        editDiv.style.display = 'none';
      }
    }

    // 页面加载时不渲染编辑区
    window.onload = function() {
      // 不自动渲染编辑区
    };
  </script>
</body>
</html>
