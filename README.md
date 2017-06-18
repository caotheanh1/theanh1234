goog.provide('farming.Land');<đất cung cấp>
goog.require('lime.Sprite');<yêu cầu của đất>

/**
 * Land elements<các yếu tố đất đai>
 * 
 * @param {} gameObj
 */
farming.Land = function(gameObj, playerObj) {
    goog.base(this);
    this.setAnchorPoint(0, 0);
    this.setSize(gameObj.tile_size,gameObj.tile_size);
    this.setFill('images/bare_land.png');
 
    this.state = this.EMPTY;
   -Các trạng thái của đất. 
    
    //user input<đầu vào của người dùng>
    var land = this;
    goog.events.listen(this,['mousedown', 'touchstart'], function(e) {
        e.event.stopPropagation();        
        if(land.state == land.EMPTY && playerObj.money >= gameObj.costPlowing) {
        -Người chơi khi thu hoạch thì sẽ có thêm tiền.
            //plow land<cày đất>    
            land.setFill('images/plowed.png')
            land.state = land.PLOWED;
            -người chơi click vào ô đất chưa cải tạo để đất được cải tạo.
            //update player money<cập nhật tiền của người chơi>
            playerObj.money -= gameObj.costPlowing;
            gameObj.updateMoney();
        }   -Hệ thống sẽ cập nhật tiền của người chơi sau khi hoạch hoặc khi mua hạt giống hoặc đào đất.
        else if(land.state == land.PLOWED && playerObj.money >= gameObj.crops[playerObj.currentCrop].cost) {
            //plant<cây>
            land.setFill('images/growing.png');
            land.state = land.GROWING;
            -Trong cửa hàng có bán các loại cây như ớt,atiso,cà tím,cà chua,rau diếp.
            //store crop and left time for it to be ready and to die<bảo quản mùa màng và thời gian còn lại để nó sẵn sàng và chết>
            land.crop = playerObj.currentCrop;
            land.ripeTime = gameObj.crops[playerObj.currentCrop].time_to_ripe * 1000;
            land.deathTime = gameObj.crops[playerObj.currentCrop].time_to_death * 1000;
            -Sau 10s mà bạn chưa thu hoạch thì cây sẽ chết.
            //update player money<cập nhật tiền của người chơi>
            playerObj.money -= gameObj.crops[playerObj.currentCrop].cost;
            gameObj.updateMoney();
        }
        else if(land.state == land.READY ) {
            //harvest<mùa gặt>
            land.setFill('images/bare_land.png');
            land.state = land.EMPTY;
            -Đợi thời gian cây mọc quả rồi thu hoạch.
            //update player money<cập nhật tiền của người chơi>
            playerObj.money += gameObj.crops[land.crop].revenue;
            gameObj.updateMoney();
        }        
    });
    
    //growing plants<cây trồng>
    dt = 1000;
    lime.scheduleManager.scheduleWithDelay(function() {
        if(this.state == this.GROWING) {            
            if(this.ripeTime <= 0) {
                this.state = this.READY;
                this.setFill('images/'+gameObj.crops[this.crop].image);
            }
            else {
                this.ripeTime -= dt;
            }
        }
        else if(this.state == this.READY) {
            if(this.deathTime <= 0) {
                this.state = this.EMPTY;
                this.setFill('images/bare_land.png');
            }
            else {
                this.deathTime -= dt;
            }
        }
    }, this, dt);
}

goog.inherits(farming.Land,lime.Sprite);

//states<tiểu bang>
farming.Land.prototype.EMPTY = 0;
farming.Land.prototype.PLOWED = 1;
farming.Land.prototype.GROWING = 2;
farming.Land.prototype.READY = 3;
