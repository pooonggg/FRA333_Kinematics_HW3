# **FRA333 Homework Assignment 3 : Static Force**
แสดงขั้นตอน แนวคิดของการตรวจคำตอบ และ ผลลัพธ์ของการตรวจสอบ
# Team Members
1. [Thitirat Santichaiyakun 6518](https://github.com/Qzider)
2. [Pongsakorn Chaikaew 6539](https://github.com/pooonggg)
# Robot
![รูปหุ่นยนต์](pic1.png)
โดยจากโจทย์จะมี MDH-Table ดังนี้ 

| a_{j-1} | alpha_{j-1} | theta_j | d_j |
|-------------|------------------|----------------|---------|
| 0.0         | 0.0°             | q1 + 180°      | d_1     |
| 0.0         | 90.0°            | q2             | 0.0     |
| a_2         | 0.0°             | q3             | 0.0     |

ความยาวของแต่ละ Link
| Parameter | Length (m) |
|-----------|------------|
| d_1   | 0.0892     |
| a_2   | -0.425     |
| a_3   | -0.39243   |
| d_4   | 0.109      |
| d_5   | 0.093      |
| d_6   | 0.082      |

สร้าง MDH Parameters โดยใช้ roboticstoolbox
```py
robot = DHRobot([
        RevoluteMDH(alpha=0, a=0, d=d1, offset=pi),
        RevoluteMDH(alpha=pi/2, a=0, d=0, offset=0),
        RevoluteMDH(alpha=0, a=-a2, d=0, offset=0)
    ], name="3DOF_Robot", tool=SE3([
        [0, 0, -1, -(a3 + d6)],
        [0, 1, 0, -d5],
        [1, 0, 0, d4],
        [0, 0, 0, 1]
    ]))
``` 
กำหนดค่า q_i ที่ใช้สำหรับการสร้างมุมของแต่ละข้อต่อ
```py
q_init = [0.0, 0.0, 0.0]  # Initial joint configuration (all joints at 0 position)
``` 

## **คำถามข้อที่ 1**
จงเขียนฟังก์ชั่นในการ Jacobian ของหุ่นยนต์ตัวนี้ให้อยู่ในฟังก์ชั่นต่อไปนี้ J_e = endEffectorJacobianHW3(q)
Function Jacobian คำนวณได้จาก



### **วิธีการตรวจสอบข้อที่ 1**
## **คำถามข้อที่ 2**
จงเขียนฟังก์ชั่นในการหาสภาวะ Singularity โดยที่ _∣∣det(J(q))∣∣ < ε_ เมื่อให้ค่า _ε_ = 0.001 และ _J*(.)_ คือเมตริกซ์จาโคเบียนที่ถูกลดรูปแล้ว
### **วิธีการตรวจสอบข้อที่ 2**
## **คำถามข้อที่ 3**
จงเขียนฟังก์ชั่นในการหา effort ของแต่ละข้อต่อเมื่อมี wrench มากระทำกับจุดกึ่งกลางของเฟรมพิกัด Fe
### **วิธีการตรวจสอบข้อที่ 3**
