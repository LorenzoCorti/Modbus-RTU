# Description
Small guide to test modbus communication with a Mitsubishi inverter with modbus library and QT.
The libmodbus documentation was written by Stéphane Raimbault https://github.com/stephane/libmodbus

# Parameter inverter
to test the communication I set these parameters in the inverter:
- Pr. 77 = 2
- Pr. 79 = 0
- Pr. 117 = 2 
- Pr. 118 = 192
- Pr. 119 = 1
- Pr. 120 = 0
- Pr. 121 = 9999
- Pr. 122 = 9999
- Pr. 123 = 9999
- Pr. 124 = 0
- Pr. 340 = 10
- Pr. 549 = 1

# communication initialization, test read command and write command
In QT after creating the InverterDriver class I used this function to initialize the modbus communication and test the read command and write command.
in this case I read three registers starting from address 1003.
The write command instead provides for the inverter to run.

    void InverterDriver::modbusTest( void )
    {
       mb_ = modbus_new_rtu("/dev/ttyUSB0", 19200, 'N', 8, 2);
       if( mb_ == NULL )
       {
           retval = -1;
           qDebug()<<__PRETTY_FUNCTION__<< "failed: modbus_new_rtu";
           return retval;
       }  
        
       timeval t;
       t.tv_sec = 0;
       t.tv_usec = 100;
       modbus_set_response_timeout( mb_, &t );
       
        if( modbus_set_slave(mb_, 1) != 0 )
        {
            qDebug()<<__PRETTY_FUNCTION__<< "failed: modbus_set_slave";
            modbus_close(mb_);
            modbus_free(mb_);
        }
        if (modbus_connect(mb_) == -1) 
        {
            qDebug()<<"Modbus not start";
            modbus_close(mb_);
            modbus_free(mb_);
            exit(1);
        }
        uint16_t tab_reg[32];
        qDebug()<<"Lettura modbus:"<<tab_reg[0]<<tab_reg[1]<<tab_reg[2]<<tab_reg[3]<<tab_reg[4];
        if( modbus_read_registers(mb_, 1003, 3, tab_reg)  == -1 )
        {
            fprintf(stderr, "%s\n", modbus_strerror(errno));
            qDebug()<<"not read";
        }
        qDebug()<<"Lettura modbus:"<<tab_reg[0]<<tab_reg[1]<<tab_reg[2]<<tab_reg[3]<<tab_reg[4];

        //set frequency to 60Hz
        uint16_t data = 6000;
        if( modbus_write_registers( mb_, 13, 1, &data); )
        {
            fprintf(stderr, "%s\n", modbus_strerror(errno));
            qDebug()<<"not write";
        }
        //RUN command
        data = = 1 << 1;    //set the forward bit
        if( modbus_write_registers( mb_, 8, 1, &data); )
        {
            fprintf(stderr, "%s\n", modbus_strerror(errno));
            qDebug()<<"not write";
        }
    }
    
    
