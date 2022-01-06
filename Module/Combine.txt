`define TimeExpire2 32'd25000000
`define GameDuration 5'd31;
module clk1Hz(clk,rst,div_clk);
input clk,rst;
output div_clk;

reg div_clk;
reg[31:0]count;

always@(posedge clk)
begin 
    if(!rst)
    begin
        count<=32'd0;
        div_clk<=1'b0;
    end
    else 
    begin 
        if(count==`TimeExpire2)
        begin
            count<=32'd0;
            div_clk<=~div_clk;
        end
        else
        begin
            count<=count+32'd1;
        end
    end
end
endmodule

module level(state1,state2,state3,pointOut);
input state1,state2,state3;
output reg [4:0]pointOut;

always@(state1 or state2 or state3)
begin
    if(state1 && state2 && state3)
    begin
        pointOut <= 5'd20;
    end
    else if(!state1 && state2 && state3)
    begin
        pointOut <= 5'd15;
    end
    else
    begin
        pointOut <= 5'd10;
    end
end
endmodule

module countDown(divClk,rst,timeNow,overFlow);
input divClk,rst;
output reg [4:0]timeNow;
output reg overFlow;
always@(posedge divClk)
begin
    if(!rst)
    begin  
        timeNow <= `GameDuration;
        overFlow = 0;
    end
    else
    begin
        if(timeNow == 0)
        begin
            overFlow <= 1;
            timeNow <= timeNow - 1;
        end
        else
            timeNow <= timeNow - 1;
    end
end
endmodule

module display(count,out1,out2);
input [4:0]count;
output reg [6:0]out1;
output reg [6:0]out2;

wire [4:0]first;
wire [4:0]second;

assign first = count / 10 ;
assign second = count % 10;

always@(count)
begin
    case(first)
        4'b0000:out1=7'b1000000;//0
        4'b0001:out1=7'b1111001;//1
        4'b0010:out1=7'b0100100;//2
        4'b0011:out1=7'b0110000;//3
        4'b0100:out1=7'b0011001;//4
        4'b0101:out1=7'b0010010;//5
        4'b0110:out1=7'b0000010;//6
        4'b0111:out1=7'b1111000;//7
        4'b1000:out1=7'b0000000;//8
        4'b1001:out1=7'b0010000;//9
    endcase
    case(second)
        4'b0000:out2=7'b1000000;//0
        4'b0001:out2=7'b1111001;//1
        4'b0010:out2=7'b0100100;//2
        4'b0011:out2=7'b0110000;//3
        4'b0100:out2=7'b0011001;//4
        4'b0101:out2=7'b0010010;//5
        4'b0110:out2=7'b0000010;//6
        4'b0111:out2=7'b1111000;//7
        4'b1000:out2=7'b0000000;//8
        4'b1001:out2=7'b0010000;//9
    endcase
end
endmodule

module main(clk,rst,s1,s2,s3,out1,out2,out3,out4);
input clk,rst,s1,s2,s3;
output [6:0]out1,out2,out3,out4;
wire toggle1Hz;
clk1Hz Clk1Hz(clk,rst,toggle1Hz);
wire [4:0]targetLevel;
level Level(s1,s2,s3,targetLevel);
wire [4:0]timeNow;
wire overFlow;
countDown CountDown(toggle1Hz,rst,timeNow,overFlow);
display LevelDisplay(targetLevel,out1,out2);
display CountDownDisplay(timeNow,out3,out4);
endmodule