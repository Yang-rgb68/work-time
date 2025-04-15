# work-time
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>工作时间打卡</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            color: #333;
            text-align: center;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .big-button {
            display: block;
            width: 100%;
            padding: 15px;
            font-size: 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            margin: 10px 0;
            cursor: pointer;
        }
        .big-button:hover {
            background-color: #45a049;
        }
        .time-input {
            font-size: 24px;
            padding: 10px;
            width: 100%;
            margin: 10px 0;
            text-align: center;
        }
        .result {
            font-size: 24px;
            margin: 20px 0;
            padding: 15px;
            background-color: #f0f0f0;
            border-radius: 5px;
            text-align: center;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>工作时间打卡</h1>
        
        <p>每天下班时，在这里输入时间：</p>
        
        <input type="time" id="endTime" class="time-input" value="16:30">
        
        <button onclick="recordTime()" class="big-button">记录今天下班时间</button>
        
        <div class="result" id="todayResult"></div>
        
        <div class="result">
            本月总工作时间: <span id="totalMinutes">0</span> 分钟
            <br>
            (约 <span id="totalHours">0</span> 小时)
        </div>
        
        <h2>打卡记录</h2>
        <table id="recordTable">
            <tr>
                <th>日期</th>
                <th>下班时间</th>
                <th>工作时间(分钟)</th>
            </tr>
        </table>
        
        <button onclick="clearData()" class="big-button" style="background-color: #f44336;">清除所有记录</button>
    </div>

    <script>
        // 工作开始时间固定为6:00
        const START_HOUR = 6;
        const START_MINUTE = 0;
        
        // 加载保存的数据
        let records = JSON.parse(localStorage.getItem('workRecords') || [];
        
        // 初始化显示
        updateDisplay();
        
        // 记录时间函数
        function recordTime() {
            const endTimeInput = document.getElementById('endTime');
            const endTimeValue = endTimeInput.value;
            
            if (!endTimeValue) {
                alert('请选择下班时间');
                return;
            }
            
            // 解析输入的时间
            const [endHour, endMinute] = endTimeValue.split(':').map(Number);
            
            // 计算工作时间（分钟）
            const startTotalMinutes = START_HOUR * 60 + START_MINUTE;
            const endTotalMinutes = endHour * 60 + endMinute;
            let workMinutes = endTotalMinutes - startTotalMinutes;
            
            // 处理跨天情况（如夜班）
            if (workMinutes < 0) {
                workMinutes += 24 * 60;
            }
            
            // 获取当前日期
            const today = new Date();
            const dateStr = today.toLocaleDateString('zh-CN');
            
            // 检查今天是否已记录
            const todayRecordIndex = records.findIndex(record => record.date === dateStr);
            
            if (todayRecordIndex >= 0) {
                // 更新现有记录
                records[todayRecordIndex] = {
                    date: dateStr,
                    endTime: endTimeValue,
                    minutes: workMinutes
                };
            } else {
                // 添加新记录
                records.push({
                    date: dateStr,
                    endTime: endTimeValue,
                    minutes: workMinutes
                });
            }
            
            // 保存数据
            localStorage.setItem('workRecords', JSON.stringify(records));
            
            // 更新显示
            updateDisplay();
            
            // 显示今天的结果
            document.getElementById('todayResult').innerHTML = `
                今天工作时间: ${workMinutes} 分钟 (约 ${Math.round(workMinutes/60*10)/10} 小时)
            `;
        }
        
        // 更新显示函数
        function updateDisplay() {
            // 计算总分钟数
            const totalMinutes = records.reduce((sum, record) => sum + record.minutes, 0);
            
            // 更新总计显示
            document.getElementById('totalMinutes').textContent = totalMinutes;
            document.getElementById('totalHours').textContent = Math.round(totalMinutes/60*10)/10;
            
            // 更新表格
            const table = document.getElementById('recordTable');
            // 清空表格（保留表头）
            while (table.rows.length > 1) {
                table.deleteRow(1);
            }
            
            // 添加记录行（按日期倒序）
            records.slice().reverse().forEach(record => {
                const row = table.insertRow();
                row.insertCell().textContent = record.date;
                row.insertCell().textContent = record.endTime;
                row.insertCell().textContent = record.minutes;
            });
        }
        
        // 清除数据函数
        function clearData() {
            if (confirm('确定要清除所有记录吗？此操作不可撤销！')) {
                localStorage.removeItem('workRecords');
                records = [];
                updateDisplay();
                document.getElementById('todayResult').innerHTML = '';
            }
        }
        
        // 设置默认时间为当前时间（方便测试）
        document.addEventListener('DOMContentLoaded', function() {
            const now = new Date();
            const hours = now.getHours().toString().padStart(2, '0');
            const minutes = now.getMinutes().toString().padStart(2, '0');
            document.getElementById('endTime').value = `${hours}:${minutes}`;
        });
    </script>
</body>
</html>
