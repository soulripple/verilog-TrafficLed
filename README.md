`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date:    16:45:57 01/27/2018 
// Design Name: 
// Module Name:    traffic_led 
// Project Name: 
// Target Devices: 
// Tool versions: 
// Description: 
//
// Dependencies: 
//
// Revision: 
// Revision 0.01 - File Created
// Additional Comments: 
//
//////////////////////////////////////////////////////////////////////////////////
module traffic_led(
    input clk_system,
	 input reset,
	 input snCar,
	 input ewCar,
	 output reg snRed,
	 output reg snGreen,
	 output reg ewRed,
	 output reg ewGreen);
	 
	 reg clk_1s;
	 reg [4:0]cnt;
	 always @ (posedge clk_system or negedge reset)
	    begin
		     if(!reset)
			      begin
			        cnt<=0;
					  clk_1s<=0;
					 end
		     else if (cnt==6'd24) //时钟每跳转25次,clk_1s跳转一次，其周期为1us
			      begin
				       clk_1s<=~clk_1s;
						 cnt<=0;
				   end
			  else 
			      begin
				       cnt<=cnt+1'b1;
			      end
		  end
   ///////////////////////////////按键控制//////////////////////////////////

    reg  key_scan1;  //按键扫描新值sn
	 reg  key_scan2;  //ew
    reg  key_scan_r1; //按键扫描旧值sn
	 reg  key_scan_r2; //ew
    wire  flag_key1; //按键扫描输出sn
	 wire  flag_key2; //ew
    
	 always @(posedge clk_system )
		 begin
           key_scan1 <= snCar; //采样按键输入电平
			  key_scan2 <= ewCar;
       end
    always @(posedge clk_system)
	    begin
           key_scan_r1 <= key_scan1;
		     key_scan_r2 <= key_scan2;
	    end
    

    assign flag_key1 = key_scan_r1 & (~key_scan1);
	 assign flag_key2 = key_scan_r2 & (~key_scan2);
        //当检测到按键有下降沿变化时，代表该按键被按下，按键有效
	 reg count_sn;
	 reg count_ew;
	 wire snCar_key;
	 wire ewCar_key;
	 reg F1,F2,F3,F4;
	 always @(posedge clk_system or negedge reset)
	     begin
            if(!reset)
            begin
                F1<=1'b1;
                F2<=1'b1;
					 F3<=1'b1;
                F4<=1'b1;
            end
            else
            begin
            F1<=flag_key1;//需要检测的引脚
            F2<=F1;
				F3<=flag_key2;
            F4<=F3;
            end
			end
				assign   snCar_key = F1 && !F2;//检测上降沿
				assign   ewCar_key = F3 && !F4;//检测上降沿
	 always @(posedge clk_system or negedge reset) //记录按键有效的次数
	     begin
		      if(!reset) 
				count_sn<=0;
				else if(snCar_key==1)
		      count_sn<=count_sn+1'b1;
				else
				count_sn<=count_sn;
		  end
	 always @(posedge clk_system or negedge reset) 
	     begin
		      if(!reset) 
				count_ew<=0;
				else if(ewCar_key==1)
		      count_ew<=count_ew+1'b1;
				else
				count_ew<=count_ew;
		  end
	 
	 reg [5:0]count1;

    always @ (posedge clk_1s or negedge reset)
	     begin
		      if(!reset)//复位后从sn方向通行开始循环
				    begin
					 snRed<=1;
					 snGreen<=0;
					 ewRed<=0;
					 ewGreen<=1;
					 count1<=0;
					 end
				else if(count_sn==1&&count_ew==0)//sn紧急通信
				    begin
					 snRed<=1;
					 snGreen<=0;
					 ewRed<=0;
					 ewGreen<=1;
					 count1<=0;
					 end
				else if(count_sn==0&&count_ew==1)//ew紧急通行
				    begin
					 snRed<=0;
					 snGreen<=1;
					 ewRed<=1;
					 ewGreen<=0;
					 count1<=0;
					 end
				else if(count1==6'd14)//sn黄灯亮（sn方向的灯同时亮）
				    begin
					 snRed<=0;
					 snGreen<=0;
					 ewRed<=0;
					 ewGreen<=1;
					 count1<=count1+1'b1;
					 end
				else if(count1==6'd19)//sn红灯亮，ew绿灯亮
				    begin
					 snRed<=0;
					 snGreen<=1;
					 ewRed<=1;
					 ewGreen<=0;
					 count1<=count1+1'b1;
					 end
				else if(count1==6'd34)//ew黄灯亮
				    begin
					 snRed<=0;
					 snGreen<=1;
					 ewRed<=0;
					 ewGreen<=0;
					 count1<=count1+1'b1;
					 end
				else if(count1==6'd39)//sn绿灯亮，循环计数从头开始
				    begin
					 snRed<=1;
					 snGreen<=0;
					 ewRed<=0;
					 ewGreen<=1;
					 count1<=0;
					 end
				else begin
				    count1<=count1+1'b1;
					 end
			end
					 
	 
endmodule


