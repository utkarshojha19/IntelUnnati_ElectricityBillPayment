module ElectricityPaymentSystem (
  input wire clk,
  input wire reset,
  input wire barcode_scanner,
  input wire touchscreen,
  input wire cash_insertion,
  input wire cheque_insertion,
  input wire upi_insertion,
  input wire dd_insertion,
  output wire [3:0] display,
  output reg [7:0] scanned_parameter1,
  output reg [7:0] scanned_parameter2,
  output wire scanning_complete,
  output reg cash_dispense,
  output reg cheque_dispense,
  output reg upi_dispense,
  output reg dd_dispense,
  output reg bill_acknowledgment
);

  // Define system states
  parameter IDLE = 3'b000;
  parameter SCANNING = 3'b001;
  parameter PAYMENT_MODE_SELECTION = 3'b010;
  parameter CASH_PAYMENT = 3'b011;

  reg [1:0] state;
  reg [7:0] amount_due;
  reg [7:0] amount_paid;
  reg [7:0] change_amount;
  reg [7:0] remaining_due_amount;
  reg [7:0] cheque_number;
  reg [2:0] denomination_index;
  wire scanning_started;
  wire [7:0] data_reg;
  reg [7:0] micr_field_1;     // Example: MICR Field 1
  reg [7:0] micr_field_2;     // Example: MICR Field 2

  // Define parameter values for denominations
  parameter [2:0] DENOM_500 = 3'b000;
  parameter [2:0] DENOM_200 = 3'b001;
  parameter [2:0] DENOM_100 = 3'b010;
  parameter [2:0] DENOM_50 = 3'b011;
  parameter [2:0] DENOM_20 = 3'b100;
  parameter [2:0] DENOM_10 = 3'b101;
  parameter [2:0] DENOM_5 = 3'b110;

  // Define parameter values for display modes
  parameter [3:0] MODE_CASH = 4'b0000;
  parameter [3:0] MODE_CHEQUE = 4'b0001;
  parameter [3:0] MODE_UPI = 4'b0010;
  parameter [3:0] MODE_DD = 4'b0011;

  // Instantiate BarcodeScanner module
  BarcodeScanner barcode_scanner_inst (
    .clk(clk),
    .reset(reset),
    .barcode_scanner(barcode_scanner),
    .scanned_parameter1(scanned_parameter1),
    .scanned_parameter2(scanned_parameter2),
    .scanning_complete(scanning_complete)
  );

  always @(posedge clk) begin
    if (reset) begin
      state <= IDLE;
      scanning_complete <= 0;
      scanned_parameter1 <= 8'd0;
      scanned_parameter2 <= 8'd0;
      amount_due <= 0;
      amount_paid <= 0;
      change_amount <= 0;
      remaining_due_amount <= 0;
      cheque_number <= 0;
      denomination_index <= 0;
      display <= MODE_CASH;  // Display the prompt for cash payment
      cash_dispense <= 0;
      cheque_dispense <= 0;
      upi_dispense <= 0;
      dd_dispense <= 0;
      bill_acknowledgment <= 0;
    end else begin
      case (state)
        IDLE:
          if (barcode_scanner && scanning_complete) begin
            state <= SCANNING;
          end
          
        // if touchscreen is pressed, go to payment mode selection
        if (touchscreen) begin
          state <= PAYMENT_MODE_SELECTION;
        end

        SCANNING:
          if (barcode_scanner) begin
            scanned_parameter1 <= scanned_parameter1 << 1;
            scanned_parameter1[0] <= 1;
          end else begin
            scanned_parameter1 <= scanned_parameter1 << 1;
            scanned_parameter1[0] <= 0;
          end
          if (scanned_parameter1 == 32) begin
            scanned_parameter1 <= 0;
            scanned_parameter2 <= scanned_parameter2 << 1;
            scanned_parameter2[0] <= 1;
          end else if (scanned_parameter2 == 32) begin
            scanning_complete <= 1;
          end

        PAYMENT_MODE_SELECTION:
          if (cash_insertion) begin
            state <= CASH_PAYMENT;
            display <= MODE_CASH;
          end else if (cheque_insertion) begin
            remaining_due_amount <= calculate_remaining_due(cheque_number);
            state <= CASH_PAYMENT;
            display <= MODE_CHEQUE;
          end else if (upi_insertion) begin
            state <= CASH_PAYMENT;
            display <= MODE_UPI;
          end else if (dd_insertion) begin
            state <= CASH_PAYMENT;
            display <= MODE_DD;
          end

        CASH_PAYMENT:
          if (cash_insertion) begin
            amount_paid <= amount_paid + 5'd5;  // Increment by the inserted cash denomination
            if (amount_paid >= amount_due) begin
              change_amount <= amount_paid - amount_due;
              remaining_due_amount <= 0;
              state <= IDLE;
              // Dispense change in denominations using cash_dispense outputs
              cash_dispense <= 0;
              if (change_amount >= 500) begin
                cash_dispense <= 4'b1000;  // Dispense 500 rupees
                change_amount <= change_amount - 500;
              end else if (change_amount >= 200) begin
                cash_dispense <= 4'b0100;  // Dispense 200 rupees
                change_amount <= change_amount - 200;
              end else if (change_amount >= 100) begin
                cash_dispense <= 4'b0010;  // Dispense 100 rupees
                change_amount <= change_amount - 100;
              end else if (change_amount >=50) begin
                cash_dispense <= 4'b0001;  // Dispense 50 rupees
                change_amount <= change_amount - 50;
              end else if (change_amount > 0) begin
                // Handle remaining change amount
                cash_dispense <= 4'b0000;  // No more denominations to dispense
                // Adjust the cash_dispense outputs accordingly
                if (change_amount >= 20) begin
                  cash_dispense[3] <= 1;  // Dispense 20 rupees
                  change_amount <= change_amount - 20;
                end
                if (change_amount >= 10) begin
                  cash_dispense[2] <= 1;  // Dispense 10 rupees
                  change_amount <= change_amount - 10;
                end
                if (change_amount >= 5) begin
                  cash_dispense[1] <= 1;  // Dispense 5 rupees
                                    change_amount <= change_amount - 5;
                end
              end
              bill_acknowledgment <= 1;
            end else begin
              // Prompt for additional cash insertion on the display
              display <= MODE_CASH;  // Display the prompt for cash payment
            end
          end
        default:
          state <= IDLE;
      endcase
    end
  end

  // Function to calculate the remaining due amount based on the cheque_number
  function [7:0] calculate_remaining_due;
    input [15:0] cheque_number;

    reg [7:0] remaining_due;

    remaining_due = 0;

    // Extract the account number and check number from the cheque number
    remaining_due = cheque_number[15:8];

    // Calculate the remaining due amount based on the account number
    case (remaining_due)
      1001: remaining_due = 100;
      1002: remaining_due = 200;
      1003: remaining_due = 300;
      1004: remaining_due = 400;
      1005: remaining_due = 500;
      default: remaining_due = 0;
    endcase

    return remaining_due;
  endfunction

endmodule

module BarcodeScanner (
  input wire clk,
  input wire reset,
  input wire barcode_scanner,
  output reg [7:0] scanned_parameter1,
  output reg [7:0] scanned_parameter2,
  output wire scanning_complete
);

  always @(posedge clk) begin
    if (reset) begin
      scanned_parameter1 <= 8'd0;
      scanned_parameter2 <= 8'd0;
      scanning_complete <= 0;
    end else begin
      if (barcode_scanner) begin
        scanned_parameter1 <= scanned_parameter1 << 1;
        scanned_parameter1[0] <= 1;
      end else begin
        scanned_parameter1 <= scanned_parameter1 << 1;
        scanned_parameter1[0] <= 0;
      end
      if (scanned_parameter1 == 32) begin
        scanned_parameter1 <= 0;
        scanned_parameter2 <= scanned_parameter2 << 1;
        scanned_parameter2[0] <= 1;
      end else if (scanned_parameter2 == 32) begin
        scanning_complete <= 1;
      end
    end
  end

endmodule
