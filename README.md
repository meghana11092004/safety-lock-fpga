# safety-lock-fpga
module safety_lock(own_con, bh, pre, own, m1, m2, opt, unlock_r,
 unlock_l, prec, ownc, m1c, m2c, ranc, err);
 input own_con, bh, pre, own, m1, m2;
 input [15:0] prec, ownc;
 input [6:0]m1c, m2c;
 input [1:0] opt;
 input [7:0] ranc;
output  reg unlock_r, unlock_l, err;
 /*own_con - owner_condition(alive or not),
 bh - banking hours, pre - president, own - owwner,
 m1/2 - manager1/2, opt - option(for virtual keypad),
 unlock_r- unlock_room, unlock_l- unlock_locker, prec- president
 code,
 ownc- owner code, m1/2c- manager1/2 code, err- error signal(if
 incorrect password)*/
 parameter OWNC = 16'd7471; //predefined owner code
 parameter PREC = 16'd9863; //predefined president code
 parameter M1C = 7'd74; //predefined manager1 code
 parameter M2C = 7'd65; //predefined manager2 code
 parameter RANC = 8'd80; //predefined random code
 // for unlocking the lockers room
 always@(pre or own or bh or m1 or m2)
 begin
 case(own_con)
 1'b0 : begin  unlock_r = ~(pre & m1 & m2);
 end
 1'b1 : begin  unlock_r = ((bh & ((own & pre) | (own &
 m1 & m2))) |
 ( ~bh & ((own & pre & (m1 | m2)))));
 end
 endcase
 end
 // for unlocking the locker
 always@(opt)
 begin
 case(opt)
 // bh + pre + own
2'b00 : if(unlock_r == 1'b1)
 begin
 if((prec == PREC) & (ownc == OWNC)) begin
 unlock_l = 1'b1;
 err=1'b0;
 end
 else begin
 unlock_l=1'b0;
 err = 1'b1;
 end
 end
 // bh + own + m1 + m2
 2'b01 : if(unlock_r == 1'b1)
 begin
 if(ownc == OWNC & m1c == M1C & m2c == M2C)begin
 unlock_l = 1'b1;
 err=1'b0;
 end
 else begin
 unlock_l=1'b0;
 err = 1'b1;
 end
 end
 // bh' +own+ prec+m1|m2
 2'b10 : if(unlock_r == 1'b1)
 begin
 if(ownc == OWNC & prec == PREC & (m1c == M1C|
 m2c == M2C)) begin
 unlock_l = 1'b1;
 err=1'b0;
 end
 else begin
 unlock_l=1'b0;
 err = 1'b1;
 end
 end
 // pre + m1 + m2 + ranc
 2'b11 : if(unlock_r == 1'b0)
 begin
 if(prec == PREC & m1c == M1C & m2c == M2C & ranc
 == RANC) begin
 unlock_l = 1'b1;
 err=1'b0;
 end
 else begin
 unlock_l=1'b0;
err = 1'b1;
 end
 end
 endcase
 end
 endmodule
