# **FRA333 Homework Assignment 3 : Static Force**
แสดงขั้นตอน แนวคิดของการตรวจคำตอบ และ ผลลัพธ์ของการตรวจสอบ
# Team Members
1. [Thitirat Santichaiyakun 6518](https://github.com/Qzider)
2. [Pongsakorn Chaikaew 6539](https://github.com/pooonggg)
# Robot
![รูปหุ่นยนต์](Image/pic1.png)
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
กำหนดค่า q_init ที่ใช้สำหรับการสร้างมุมของแต่ละข้อต่อ
```py
q_init = [0.0, 0.0, 0.0]  # Initial joint configuration (all joints at 0 position)
``` 

## **คำถามข้อที่ 1**
จงเขียนฟังก์ชั่นในการ Jacobian ของหุ่นยนต์ตัวนี้ให้อยู่ในฟังก์ชั่นต่อไปนี้ J_e = endEffectorJacobianHW3(q)
โดย Function Jacobian คำนวณได้จาก
![Jacobian](Image/Jacobian.png)
ซึ่งจากหุ่นในรูปนั้นเป็นแบบ Revolute เราจึงใช้วิธีการคำนวณแบบ Revolute ซึ่งจะเขียนเป็น Function ได้ดังต่อไปนี้
```py
def endEffectorJacobianHW3(q:list[float])->list[float]:
    R,P,R_e,p_e = FKHW3(q)
    n_joints = len(q)
    J = np.zeros((6, n_joints))
    
    for i in range(n_joints):
        p_i = P[:,i]
        z = R[:,2,i] # เอาแค่ Rotation ของ Z มาใช้
        J[:3,i] = (np.cross(z,(p_e - p_i))) @ R_e # linear velocity
        J[3:,i] = z @ R_e# angular velocity
        
    return J
```
### **วิธีการตรวจสอบข้อที่ 1**
ตรวจสอบโดยใช้ MDH Parameters ที่สร้างไว้ข้างต้นและใช้ robot.jacobe ใน robotictoolsbox เทียบกับ endEffectorJacobianHW3
```py
# Calculate Jacobian using custom function and Robotics Toolbox
    J_custom = endEffectorJacobianHW3(q_init)
    J_rtb = robot.jacobe(q_init)
```
### ผลลัพธ์ที่ได้เมื่อเทียบกัน
```py
==================================================
ตรวจคำตอบข้อ 1: เปรียบเทียบ Jacobian
==================================================

Jacobian (คำนวณเอง):
----------------------------------------
  0.8994  -0.0000  -0.0000
 -0.0000  -0.8994  -0.4744
 -0.1090  -0.0930  -0.0930
  0.0000   1.0000   1.0000
  1.0000  -0.0000  -0.0000
 -0.0000   0.0000   0.0000

Jacobian (Robotics Toolbox):
----------------------------------------
  0.8994   0.0000   0.0000
 -0.0000  -0.8994  -0.4744
 -0.1090  -0.0930  -0.0930
  0.0000   1.0000   1.0000
  1.0000   0.0000   0.0000
  0.0000   0.0000   0.0000
```
จากผลลัพธ์ของทั้งสอง Function จะเห็นได้ว่ามีค่า Jacobian Matrix ที่ตรงกัน
## **คำถามข้อที่ 2**
จงเขียนฟังก์ชั่นในการหาสภาวะ Singularity โดยที่ _∣∣det(J(q))∣∣ < ε_ เมื่อให้ค่า _ε_ = 0.001 และ _J*(.)_ คือเมตริกซ์จาโคเบียนที่ถูกลดรูปแล้ว
```py
def checkSingularityHW3(q:list[float])->bool:
    # คำนวณ Jacobian
    J = endEffectorJacobianHW3(q)
    
    # ใช้เฉพาะส่วนของ Jacobian ที่เป็น 3x3 (เชิงเส้น)
    J_reduced = J[:3, :]  # ลดรูป Jacobian สำหรับตรวจหา Singularity

    # คำนวณ determinant ของ Jacobian ที่ลดรูปแล้ว
    det_J = np.linalg.det(J_reduced)
    # คำนวณหาค่า norm จาก det_J
    Norm_J = np.linalg.norm(det_J)

    # ใช้ค่า Norm เพื่อตรวจสอบ Singularity
    if  Norm_J < epsilon:
        return 1  # Singularity
    else:
        return 0  # ปกติ
```
โดยนำค่า Jacobian ที่ได้จากข้อที่ 1 มาทำการลดรูปให้เหลือแค่ในส่วนของ linear velocity เพื่อนำมาคำนวณหา det และทำการหา norm โดยได้ Function np.linalg.det และ np.linalg.norm ตามลำดับ และ check ค่า singularity โดยการกำหนดขอบเขตโดยใช้  _ε_ เป็นตัวกำหนดขอบเขต
### **วิธีการตรวจสอบข้อที่ 2**
สร้าง q_list กำหนดให้ 3 ตัวแรกเป็น q ที่ทำให้หุ่นอยู่ในสภาวะปกติและตัวที่เหลือคือค่าที่อยู่ใกล้สภาวะ singularity มาตรวจ check ว่าเป็น singularity หรือไม่ โดยทำการ check โดยใช้ abs ใน numpy มาเช็คค่า norm แล้วทำการกำหนดขอบเขตเช่นเดิม
```py
q_list = [
        [0.0, -pi/4, -pi/4],
        [0.0, pi/4, pi/2],
        [0.0, 0.0, -pi/2],
        [-0.24866892, 0.22598268, -0.19647569],
        [-1.74, -1.06, -1.15],
        [0.0 , -pi/2 , -0.2]
    ]
    
    print(f"{'Configuration':<15} {'คำนวณเอง':<15} {'Robotics Toolbox':<20} {'Singularity Value (คำนวณเอง)':<30} {'Singularity Value (RTB)':<30}")
    print("-" * 110)
    
    for i, q in enumerate(q_list):
        # Check singularity using custom function
        flag_custom = checkSingularityHW3(q)
        J_custom = endEffectorJacobianHW3(q)
        J_custom_linear = J_custom[:3, :]  # Extract linear velocity components
        S_custom = abs(np.linalg.det(J_custom_linear))  # Calculate determinant

        # Check singularity using Robotics Toolbox
        J_rtb = robot.jacobe(q)
        J_rtb_linear = J_rtb[:3, :]  # Extract linear velocity components
        S_rtb = abs(np.linalg.det(J_rtb_linear))  # Calculate determinant
        flag_rtb = 1 if S_rtb < 0.001 else 0  # Threshold for singularity detection
```
### ผลลัพธ์ที่ได้เมื่อเทียบกัน
```py
ตรวจคำตอบข้อ 2: ตรวจสอบ Singularity
==================================================
Configuration   คำนวณเอง        Robotics Toolbox     Singularity Value (คำนวณเอง)   Singularity Value (RTB)       
--------------------------------------------------------------------------------------------------------------
Config 1         0               0                    0.045108                       0.045108
Config 2         0               0                    0.020307                       0.020307
Config 3         0               0                    0.104446                       0.104446
Config 4         1               1                    0.000529                       0.000529
Config 5         1               1                    0.000103                       0.000103
Config 6         1               1                    0.000004                       0.000004

```
จากผลลัพธ์ที่ได้เมื่อเทียบกับทั้งสอง Function มีผลลัพธ์ที่ตรงกัน
## **คำถามข้อที่ 3**
จงเขียนฟังก์ชั่นในการหา effort ของแต่ละข้อต่อเมื่อมี wrench มากระทำกับจุดกึ่งกลางของเฟรมพิกัด Fe
คำนวณหา effort จากสูตร 

$$\tau\ =\ J^T \times w$$

โดยที่กำหนดค่า w_init ที่ดเป็นค่า wrench 
```py
w_init = [0.0, 10.0, 0.0, 0.0, 0.0, 0.0]  # Initial end-effector wrench (force and moment)
```
สร้าง Function tau = computeEffortHW3(q,w)
```py
def computeEffortHW3(q:list[float], w:list[float])->list[float]:
    # Jacobian ของหุ่นยนต์
    J = endEffectorJacobianHW3(q)
    # คำนวณ Jacobian Transpose
    J_T = np.transpose(J)
    # คำนวณค่า torque สำหรับข้อต่อ
    tau = J_T @ w  
    return tau
```
ทำการหา Torque โดยใช้จาก Function ข้อที่ 1 มาทำการ Transpose หลังจากนั้นนำมาคูณกับค่า wrench ที่กำหนดไว้เพื่อหา Torque
### **วิธีการตรวจสอบข้อที่ 3**
ใช้ Function ใน roboticstoolbox เพื่อทำการตรวจสอบว่าค่า Force จากการคำนวณย้อนกลับนั้นตรงกับค่า Force ใน Wrench ที่กำหนดหรือไม่
```py
effort_custom = computeEffortHW3(q_init, w_init)
    J = robot.jacobe(q_init)
    effort_rtb = robot.pay(W=w_init, J=J)  # Use pay() method for inverse dynamics

    print("Joint Efforts:")
    print(f"{'Joint':<10} {'คำนวณเอง':<15} {'Robotics Toolbox':<20}")
    print("-" * 45)
    for i in range(3):
        print(f"Joint {i+1:<5} {effort_custom[i]:15.4f} {effort_rtb[i]:20.4f}")

    def calc_forces(effort):
        """
        Calculate end-effector forces from joint efforts.
        
        Args:
        effort (list): Joint efforts
        
        Returns:
        list: Calculated end-effector forces
        
        This function uses the robot's geometry to convert joint efforts to end-effector forces.
        """
        return [
            effort[0] / (a2 + a3 + d6),  # Force in x-direction
            effort[1] / (a2 + a3 + d6),  # Force in y-direction
            effort[2] / (a3 + d6)        # Force in z-direction
        ]

    forces_custom = calc_forces(effort_custom)
    forces_rtb = calc_forces(effort_rtb)
```
### ผลลัพธ์ที่ได้เมื่อเทียบกัน
```py
ตรวจคำตอบข้อ 3: คำนวณ Effort และแรงที่ปลายมือ
==================================================
Joint Efforts:
Joint      คำนวณเอง        Robotics Toolbox
---------------------------------------------
Joint 1              0.0000               0.0000
Joint 2              8.9943               8.9943
Joint 3              4.7443               4.7443

แรงที่ปลายมือ (N):
Direction  คำนวณเอง        Robotics Toolbox
---------------------------------------------
f1                  0.0000               0.0000
f2                 10.0000              10.0000
f3                 10.0000              10.0000
```
จากผลลัพธ์ที่ได้เมื่อเทียบกับทั้งสอง Function มีผลลัพธ์ที่ตรงกัน
