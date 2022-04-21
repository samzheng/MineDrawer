<template>
    <view>
        <canvas style="width: 1920px; height: 1080px;" canvas-id="canvas" id="canvas"></canvas>
    </view>
</template>

<script>
	export default {
		data() {
			return {
                mPageWidthRate: 0.95,
                mPageHeightRate: 0.95,
                mScreenPaddingLeftRate: 0.05,
                mScreenPaddingRightRate: 0.05,
                mScreenPaddingTopRate: 0.1,
                mScreenPaddingBottomRate: 0.1,
                mScreenMiddleRate: 0.2,

                mRailwayRate: 0.4,
                mTrackHeightRateInRailway: 0.8,

                mCarWidth: 80,
                mCarCircleRadius: 30,
                mCarMarginX: 30,

                mTrackColor: 'rgb(0,86,155)',
                mCarFillColor: 'rgb(0, 162, 233)',
                mTrackLineWidth: 5,
                mCarLineWidth: 2,
                mCarNameColor: '#fff',
                mCarNamePadding: 10,
                mCarNameFont: '15px serif',
                mCarToneFont: '15px serif',
                mCarToneColor: '#fff',
                mCarNameMarginTop: 3,
                mCarCirlcleColor: 'rgb(241,133,23)',

                mTableHeaderMarginBottom: 30,
                mTableHeaderColor: 'rgb(0, 86, 155)',
                mTableHeaderFont: 'bold 20px serif',
                mTableBorderColor: '#000',
                mTableBorderWidth: 2,

                mTableColumnWidth: 100,
                mTableRowHeight: 40,

                mLeftTableColumn0Color: '#fff',
                mLeftTableColumn0Font: 'bold 15px serif',
                mTableLowTemperatureColor: 'rgb(0,155,76)',
                mTableMidTemperatureColor: 'rgb(241,133,23)',
                mTableHighTemperatureColor: 'rgb(230,32,40)',

                mTableMargin: 80,
                mRightTableColumn0Color: 'rgb(0, 86, 155)',
                mRightTableColumn0Font: '12px serif',

                mLowStatusStyle: '#0f0',
                mMidStatusStyle: '#ff0',
                mHighStatusStyle: '#f00',
                mStatusRadius: 12,

                context: null,
                screenWidth: 1920,
                screenHeight: 1080,
                paddingLeft: 0,
                paddingRight: 0,
                paddingTop: 0,
                paddingBottom: 0,
                paddingMiddle: 0,
                railwayWidth: 0,
                railwayHeight: 0,
                carList: []
			}
		},
        onReady: function (e) {
            this.context = uni.createCanvasContext('canvas');
            this.startDrawApi();
        },
		onLoad() {
			// #ifdef APP-PLUS
			plus.screen.lockOrientation('landscape-primary');
			// #endif
		},
		methods: {
			startDrawApi() {
				var self = this;
				uni.request({
				    url: 'http://106.54.170.187/Frings/Role/BindData', //仅为示例，并非真实接口地址。
				    data: {
				        
				    },
				    success: (res) => {
				        if (res.data.length == 1 && res.data[0].ID == 1) {
							self.startDraw();
						}
				    }
				});
			},
            startDraw() {
                this.initializeVariables();
                this.gameLoop();
            },
            initializeVariables() {
                this.paddingLeft = Math.round(this.screenWidth * this.mScreenPaddingLeftRate);
                this.paddingRight = Math.round(this.screenWidth * this.mScreenPaddingRightRate);;
                this.paddingTop = Math.round(this.screenHeight * this.mScreenPaddingTopRate);
                this.paddingBottom = Math.round(this.screenHeight * this.mScreenPaddingBottomRate);
                this.paddingMiddle = Math.round(this.screenHeight * this.mScreenMiddleRate);
                this.railwayWidth = this.screenWidth - this.paddingLeft - this.paddingRight;
                this.railwayHeight = Math.round((this.screenHeight - this.paddingTop - this.paddingBottom - this.paddingMiddle) * this.mRailwayRate);
                this.tableHeight = this.screenHeight - Math.round((this.screenHeight - this.paddingTop - this.paddingBottom - this.paddingMiddle) * (1 - this.mRailwayRate));
            },
            gameLoop() {
                setTimeout(this.gameLoop, 1000);

                this.fetchCarList();
                this.drawScreen();
                this.refreshCarsData();
            },
            refreshCarsData() {
                for(var i = 0; i < this.carList.length; i ++) {
                    var car = this.carList[i];
                    this.refreshCarData(car);
                }
            },
            refreshCarData(car) {
                car.Data1 = this.getTempTemperature();
                car.Data2 = this.getTempTemperature();
                car.Data3 = this.getTempTemperature();
                car.Data4 = this.getTempTemperature();
                car.Data5 = this.getTempTemperature();

                car.Status1 = this.getTempStatus();
                car.Status2 = this.getTempStatus();
                car.Status3 = this.getTempStatus();
                car.Status4 = this.getTempStatus();
                car.Status5 = this.getTempStatus();
            },
            drawScreen() {
                this.context.fillStyle = '#eee';
                this.context.fillRect(0, 0, this.screenWidth, this.screenHeight);

                this.drawTrack();
                this.drawCarsInLine();
                this.drawLeftTable();
                this.drawMiddleTable();
                this.drawRightTable();
                this.context.draw();
            },
            drawTrack() {
                var carHeight = this.railwayHeight;
                var trackHeight = Math.round(carHeight * this.mTrackHeightRateInRailway);
                var marginY = Math.round(carHeight - trackHeight / 2);
                //
                var topTrackX = this.paddingLeft;
                var topTrackY = this.paddingTop + marginY;
                var bottomTrackX = this.paddingLeft;
                var bottomTrackY = this.paddingTop + this.railwayHeight - marginY;
                //
                this.context.strokeStyle = this.mTrackColor;
                //this.context.setStrokeStyle(this.mTrackColor);
                this.context.lineWidth = this.mTrackLineWidth;
                //this.context.setLineWidth(this.mTrackLineWidth);
                this.context.moveTo(topTrackX, topTrackY);
                this.context.lineTo(topTrackX + this.railwayWidth, topTrackY);
                this.context.stroke();

                this.context.moveTo(bottomTrackX, bottomTrackY);
                this.context.lineTo(bottomTrackX + this.railwayWidth, bottomTrackY);
                this.context.stroke();
            },
            drawCarsInLine() {
                var carHeight = this.railwayHeight;
                var trackHeight = Math.round(carHeight * this.mTrackHeightRateInRailway);
                var marginY = Math.round(carHeight - trackHeight / 2);

                var topTrackX = this.paddingLeft;
                var topTrackY = this.paddingTop + marginY;
                var bottomTrackX = this.paddingLeft;
                var bottomTrackY = this.paddingTop + this.railwayHeight - marginY;

                var x = topTrackX + this.mCarMarginX;
                var y = this.paddingTop;

                for(var i = 0; i < this.carList.length; i ++) {
                    var car = this.carList[i];
                    this.context.fillStyle = this.mCarFillColor;
                    this.context.lineWidth = this.mCarLineWidth;
                    this.context.fillRect(x, y, this.mCarWidth, carHeight);

                    var wordX = x + this.mCarNamePadding;
                    var wordY = y + this.mCarLineWidth + this.mCarNameMarginTop;
                    this.context.textBaseline = 'top';
                    this.context.textAlign = 'start';
                    this.context.fillStyle = this.mCarNameColor;
                    this.context.font = this.mCarNameFont;
                    this.context.fillText(car.Name, wordX, wordY);

                    this.context.lineWidth = 1;
                    var circleX = x + Math.round(this.mCarWidth/2);
                    var circleMargin = Math.round((this.mCarWidth - this.mCarMarginX)/2);
                    var circleY = y + carHeight - circleMargin - this.mCarCircleRadius;

                    this.context.beginPath();
                    this.context.fillStyle = this.mCarCirlcleColor;
                    this.context.arc(circleX, circleY, this.mCarCircleRadius, 0, Math.PI * 360 / 180, false);
                    this.context.fill();
                    this.context.closePath();

                    this.context.fillStyle = this.mCarToneColor;
                    this.context.font = this.mCarToneFont;
                    this.context.textAlign = 'center';
                    this.context.fillText(car.Tone, circleX, circleY);
                    x += this.mCarWidth + this.mCarMarginX;
                }
            },
            drawLeftTable() {
                var x = this.paddingLeft;
                var contentTotalHeight = Math.round(this.screenHeight * (1 - this.mScreenPaddingTopRate - this.mScreenPaddingBottomRate - this.mScreenMiddleRate));
                var talbeHeight = Math.round(contentTotalHeight * (1 - this.mRailwayRate));
                var paddingBottom = Math.round(this.screenHeight * this.mScreenPaddingBottomRate);
                var y = paddingBottom + talbeHeight;

                this.context.fillStyle = this.mTableHeaderColor;
                this.context.font = this.mTableHeaderFont;
                this.context.textAlign = 'start';
                this.context.fillText('温度显示', x, y);

                y += this.mTableHeaderMarginBottom;
                var tableWidth = (1 + this.carList.length) * this.mTableColumnWidth;
                var tableHeight = 6 * this.mTableRowHeight;

                var tableX = x;
                var tableY = y;

                this.context.fillStyle = this.mTableHeaderColor;
                this.context.fillRect(tableX, tableY + this.mTableBorderWidth, this.mTableColumnWidth, this.mTableRowHeight * 6);

                this.context.strokeStyle = this.mTableBorderColor;
                this.context.lineWidth = this.mTableBorderWidth;
                for(var i = 0; i < 7; i ++) {
                    y = tableY + (i * this.mTableRowHeight + this.mTableBorderWidth);
                    this.context.beginPath();
                    this.context.moveTo(tableX, y);
                    this.context.lineTo(tableX + tableWidth, y);
                    this.context.stroke();
                    this.context.closePath();
                }

                for(var i = 0; i < this.carList.length + 2; i ++) {
                    x = tableX + (this.mTableColumnWidth) * i;
                    y = tableY + (this.mTableRowHeight) * 6 + this.mTableBorderWidth;
                    this.context.moveTo(x, tableY + this.mTableBorderWidth);
                    this.context.lineTo(x, y);
                    this.context.stroke();
                }

                for(var columnIndex = 0; columnIndex < this.carList.length + 1; columnIndex ++) {
                    if (columnIndex == 0) {
                        this.writeLeftTableColumn0Data(tableX, tableY);
                    } else {
                        this.writeLeftTableColumnData(columnIndex, tableX, tableY);
                    }
                }
            },
            writeLeftTableColumn0Data(tableX, tableY) {
                this.context.fillStyle = this.mLeftTableColumn0Color;
                this.context.font = this.mLeftTableColumn0Font;
                this.context.textBaseline = 'middle';
                this.context.textAlign = 'center';
                for(var rowIndex = 0; rowIndex < 6; rowIndex ++) {
                    var text = (rowIndex == 0) ? '车号' : '****';
                    this.context.fillText(text, tableX + this.mTableColumnWidth / 2, tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight));
                }
            },
            writeLeftTableColumnData(columnIndex, tableX, tableY) {
                var car = this.carList[columnIndex - 1];
                this.context.fillStyle = this.mLeftTableColumn0Color;
                this.context.font = this.mLeftTableColumn0Font;
                this.context.textBaseline = 'middle';
                this.context.textAlign = 'center';
                for(var rowIndex = 0; rowIndex < 6; rowIndex ++) {
                    var text = this.getCarData(car, rowIndex) + '℃';
                    if (rowIndex == 0) {
                        text = car.Name;
                        this.context.fillStyle = this.mTableHeaderColor;
                        this.context.font = this.mLeftTableColumn0Font;
                    } else {
                        this.context.fillStyle = this.getCarFillStyle(car, rowIndex);
                        this.context.font = this.getCarFont(car, rowIndex);
                    }

                    this.context.fillText(text, tableX + this.mTableColumnWidth / 2 + (columnIndex * this.mTableColumnWidth), tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight));
                }
            },
            getCarFont(car, rowIndex) {
                if (rowIndex == 0) {
                    return '12px serif';
                } else {
                    return '18px serif';
                }
            },
            getCarData(car, rowIndex) {
                switch(rowIndex) {
                    case 0:
                        return car.Name;
                    case 1:
                        return car.Data1;
                    case 2:
                        return car.Data2;
                    case 3:
                        return car.Data3;
                    case 4:
                        return car.Data4;
                    case 5:
                        return car.Data5;
                    default:
                        return '-';
                }
            },
            getCarFillStyle(car, rowIndex) {
                switch(rowIndex) {
                    case 0:
                        return this.mLeftTableColumn0Color;
                    case 1:
                        return this.getCarFillStyleByData(car.Data1);
                    case 2:
                        return this.getCarFillStyleByData(car.Data2);
                    case 3:
                        return this.getCarFillStyleByData(car.Data3);
                    case 4:
                        return this.getCarFillStyleByData(car.Data4);
                    case 5:
                        return this.getCarFillStyleByData(car.Data5);
                    default:
                        return this.mLeftTableColumn0Color;
                }
            },
            getCarFillStyleByData(data) {
                if (data < 60) {
                    return this.mTableLowTemperatureColor;
                } else if (data < 80) {
                    return this.mTableMidTemperatureColor;
                } else {
                    return this.mTableHighTemperatureColor;
                }
            },
            getCarFillStyleByData(data) {
                if (data < 60) {
                    return this.mTableLowTemperatureColor;
                } else if (data < 80) {
                    return this.mTableMidTemperatureColor;
                } else {
                    return this.mTableHighTemperatureColor;
                }
            },
            drawMiddleTable() {
                var x = this.paddingLeft + ((1 + this.carList.length) * this.mTableColumnWidth) + this.mTableMargin;
                var contentTotalHeight = Math.round(this.screenHeight * (1 - this.mScreenPaddingTopRate - this.mScreenPaddingBottomRate - this.mScreenMiddleRate));
                var talbeHeight = Math.round(contentTotalHeight * (1 - this.mRailwayRate));
                var paddingBottom = Math.round(this.screenHeight * this.mScreenPaddingBottomRate);
                var y = paddingBottom + talbeHeight;

                this.context.fillStyle = this.mTableHeaderColor;
                this.context.font = this.mTableHeaderFont;
                this.context.textAlign = 'start';
                this.context.fillText('DI显示', x, y);

                y += this.mTableHeaderMarginBottom;
                var tableWidth = (1 + this.carList.length) * this.mTableColumnWidth;
                var tableHeight = 6 * this.mTableRowHeight;

                var tableX = x;
                var tableY = y;

                this.context.fillStyle = this.mTableHeaderColor;
                this.context.fillRect(tableX, tableY + this.mTableBorderWidth, this.mTableColumnWidth, this.mTableRowHeight * 6);

                this.context.strokeStyle = this.mTableBorderColor;
                this.context.lineWidth = this.mTableBorderWidth;
                for(var i = 0; i < 7; i ++) {
                    y = tableY + (i * this.mTableRowHeight + this.mTableBorderWidth);
                    this.context.beginPath();
                    this.context.moveTo(tableX, y);
                    this.context.lineTo(tableX + tableWidth, y);
                    this.context.stroke();
                    this.context.closePath();
                }

                for(var i = 0; i < this.carList.length + 2; i ++) {
                    x = tableX + (this.mTableColumnWidth) * i;
                    y = tableY + (this.mTableRowHeight) * 6 + this.mTableBorderWidth;
                    this.context.moveTo(x, tableY + this.mTableBorderWidth);
                    this.context.lineTo(x, y);
                    this.context.stroke();
                }

                for(var columnIndex = 0; columnIndex < this.carList.length + 1; columnIndex ++) {
                    if (columnIndex == 0) {
                        this.writeLeftTableColumn0Data(tableX, tableY);
                    } else {
                        this.writeMiddleTableColumnData(columnIndex, tableX, tableY);
                    }
                }
            },
            writeLeftTableColumn0Data(tableX, tableY) {
                this.context.fillStyle = this.mLeftTableColumn0Color;
                this.context.font = this.mLeftTableColumn0Font;
                this.context.textBaseline = 'middle';
                this.context.textAlign = 'center';
                for(var rowIndex = 0; rowIndex < 6; rowIndex ++) {
                    var text = (rowIndex == 0) ? '车号' : '****';
                    this.context.fillText(text, tableX + this.mTableColumnWidth / 2, tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight));
                }
            },
            writeMiddleTableColumnData(columnIndex, tableX, tableY) {
                var car = this.carList[columnIndex - 1];
                this.context.fillStyle = 'rgb(0,86,155)';
                this.context.font = this.mLeftTableColumn0Font;
                this.context.textBaseline = 'middle';
                this.context.textAlign = 'center';
                for(var rowIndex = 0; rowIndex < 6; rowIndex ++) {
                    var text = '100101010';
                    if (rowIndex == 0) {
                        text = car.Name;
                    } else {
                        this.context.font = ' 15px serif';
                    }
                    this.context.fillText(text, tableX + this.mTableColumnWidth / 2 + (columnIndex * this.mTableColumnWidth), tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight));
                }
            },
            drawRightTable() {
                var x = this.paddingLeft + (1 + this.carList.length) * this.mTableColumnWidth * 2 + this.mTableMargin * 2;
                var contentTotalHeight = Math.round(this.screenHeight * (1 - this.mScreenPaddingTopRate - this.mScreenPaddingBottomRate - this.mScreenMiddleRate));
                var talbeHeight = Math.round(contentTotalHeight * (1 - this.mRailwayRate));
                var paddingBottom = Math.round(this.screenHeight * this.mScreenPaddingBottomRate);
                var y = paddingBottom + talbeHeight;

                this.context.fillStyle = this.mTableHeaderColor;
                this.context.font = this.mTableHeaderFont;
                this.context.textAlign = 'start';
                this.context.fillText('设备状态管理', x, y);

                y += this.mTableHeaderMarginBottom;
                var tableWidth = (1 + this.carList.length) * this.mTableColumnWidth;
                var tableHeight = 5 * this.mTableRowHeight;

                var tableX = x;
                var tableY = y;

                this.context.fillStyle = this.mTableHeaderColor;
                this.context.fillRect(tableX, tableY + this.mTableBorderWidth, this.mTableColumnWidth, this.mTableRowHeight * 6);

                this.context.strokeStyle = this.mTableBorderColor;
                this.context.lineWidth = this.mTableBorderWidth;
                for(var i = 0; i < 7; i ++) {
                    y = tableY + (i * this.mTableRowHeight + this.mTableBorderWidth);
                    this.context.moveTo(tableX, y);
                    this.context.lineTo(tableX + tableWidth, y);
                    this.context.stroke();
                }

                for(var i = 0; i < this.carList.length + 2; i ++) {
                    if (i == 0 || i == this.carList.length + 1) {
                        y = tableY + (this.mTableRowHeight) * 6 + this.mTableBorderWidth;
                    } else {
                        y = tableY + (this.mTableRowHeight) * 5 + this.mTableBorderWidth;
                    }
                    if (i == 1) {
                        y = tableY + (this.mTableRowHeight) * 6 + this.mTableBorderWidth;
                    }
                    x = tableX + (this.mTableColumnWidth) * i;
                    this.context.moveTo(x, tableY + this.mTableBorderWidth);
                    this.context.lineTo(x, y);
                    this.context.stroke();
                }

                for(var columnIndex = 0; columnIndex < this.carList.length + 1; columnIndex ++) {
                    if (columnIndex == 0) {
                        this.writeRightTableColumn0Data(tableX, tableY);
                    } else {
                        this.writeRightTableColumnData(columnIndex, tableX, tableY);
                    }
                    if (columnIndex == 1) {
                        this.writeZhongJiQiStatus(tableX, tableY);
                    }
                }
            },
            writeRightTableColumn0Data(tableX, tableY) {
                this.context.fillStyle = '#fff';
                this.context.font = this.mRightTableColumn0Font;
                this.context.textBaseline = 'middle';
                this.context.textAlign = 'center';
                this.context.font = 'serif'
                for(var rowIndex = 0; rowIndex < 7; rowIndex ++) {
                    var text = this.getRightTableColumn0Caption(rowIndex);
                    this.context.fillText(text, tableX + this.mTableColumnWidth / 2, tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight));
                }
            },
            getRightTableColumn0Caption(rowIndex) {
                switch (rowIndex) {
                    case 0:
                        return '车号';
                    case 1:
                        return '大车位置采集器';
                    case 2:
                        return 'DI采集器'
                    case 3:
                        return 'AI采集器';
                    case 4:
                        return '温度采集器';
                    case 5:
                        return '公共设备';
                    default:
                        return '---';
                }
            },
            writeRightTableColumnData(columnIndex, tableX, tableY) {
                var car = this.carList[columnIndex - 1];

                for(var rowIndex = 0; rowIndex < 5; rowIndex ++) {
                    if (rowIndex == 0) {
                        var text = car.Name;
                        this.context.fillStyle = this.mRightTableColumn0Color;
                        this.context.font = this.mLeftTableColumn0Font;
                        this.context.textBaseline = 'middle';
                        this.context.textAlign = 'center';
                        this.context.font = this.mLeftTableColumn0Font;
                        this.context.fillText(text, tableX + this.mTableColumnWidth / 2 + (columnIndex * this.mTableColumnWidth), tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight));
                    } else {
                        this.context.beginPath();
                        var status = this.getCarStatus(car, rowIndex);
                        this.context.fillStyle = this.getStatusStyle(status);
                        this.context.arc(tableX + this.mTableColumnWidth / 2 + (columnIndex * this.mTableColumnWidth), tableY + this.mTableRowHeight / 2 + (rowIndex * this.mTableRowHeight), this.mStatusRadius, 0, Math.PI * 360 / 180, false);
                        this.context.fill();
                        this.context.closePath();
                    }
                }
            },
            getCarStatus(car, sequence) {
                switch (sequence) {
                    case 1:
                        return car.Status1;
                    case 2:
                        return car.Status2;
                    case 3:
                        return car.Status3;
                    case 4:
                        return car.Status4;
                    case 5:
                        return car.Status5;
                    default:
                        return -1;
                }
            },
            writeZhongJiQiStatus(tableX, tableY) {
                var columnIndex = 1;
                var rowIndex = 5;

                var wordMargin = 10;

                var x = tableX + this.mTableColumnWidth + wordMargin;
                var y = tableY + 5.5 * this.mTableRowHeight;

                this.context.fillStyle = 'rgb(0,86,155)';
                this.context.textBaseline = 'middle';
                this.context.textAlign = 'start';
                this.context.fillText('网间协调器', x, y);
                //this.context.fillText('网络中继器', x + 2 * this.mTableColumnWidth, y);



                x += this.context.measureText('网间协调器').width + wordMargin + this.mStatusRadius;

                this.context.beginPath();
                var status = this.getTempStatus();
                this.context.fillStyle = this.getStatusStyle(status);
                this.context.arc(x, y, this.mStatusRadius, 0, Math.PI * 360 / 180, false);
                this.context.fill();
                this.context.closePath();

                x += this.mStatusRadius + wordMargin;
                this.context.fillStyle = 'rgb(0,86,155)';
                this.context.fillText('网络中继器', x, y);

                x +=  this.context.measureText('网络中继器').width + wordMargin + this.mStatusRadius;

                this.context.beginPath();
                var status = this.getTempStatus();
                this.context.fillStyle = this.getStatusStyle(status);
                this.context.arc(x, y, this.mStatusRadius, 0, Math.PI * 360 / 180, false);
                this.context.fill();
                this.context.closePath();
            },
            getStatusStyle(status) {
                switch (status) {
                    case 1:
                        return this.mLowStatusStyle;
                    case 2:
                        return this.mMidStatusStyle;
                    case 3:
                        return this.mHighStatusStyle;
                    default:
                        return this.mHighStatusStyle;
                }
            },
            fetchCarList() {
                if (this.carList.length == 0) {
                    this.addCar();
                    this.addCar();
                    this.addCar();
                    this.addCar();
                }
            },
            addCar() {
                var name = Math.floor(Math.random() * 1000) + 1;
                var tone = (Math.floor(Math.random() * 10) + 990) / 10;

                var car = {
                    Name: name + '#车',
                    Tone: tone + 'T',
                    Data1: this.getTempTemperature(),
                    Data2: this.getTempTemperature(),
                    Data3: this.getTempTemperature(),
                    Data4: this.getTempTemperature(),
                    Data5: this.getTempTemperature(),
                    Status1: this.getTempStatus(),
                    Status2: this.getTempStatus(),
                    Status3: this.getTempStatus(),
                    Status4: this.getTempStatus(),
                    Status5: this.getTempStatus()
                };

                this.carList.push(car);
            },
            getTempStatus() {
                var rd = Math.floor(Math.random() * 10);
                if (rd < 7) {
                    return 1;
                } else if (rd < 9) {
                    return 2;
                } else {
                    return 3;
                }
            },
            getTempTemperature() {
                var rd = Math.floor(Math.random() * 10);
                if (rd < 6) {
                    return 30 + Math.floor(Math.random() * 30);
                } else if (rd < 8) {
                    return 60 + Math.floor(Math.random() * 20);
                } else {
                    return 80 + Math.floor(Math.random() * 10);
                }
            }
		}
	}
</script>

<style>
</style>
