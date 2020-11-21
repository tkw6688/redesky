//请勿修改配置格式
const SerIp = "redesky.com", SerPor = 25565, NPort = 25565;
//源服务器ip 源服务器端口 转发到端口
const DVisit = true, DisIp = "", DESuf = [];
//伪装访问(true或false) 伪装访问的ip地址(留空将使用源服务器ip) 伪装后缀(通常不需要修改,非专业请勿修改)
const OLog = false;
//记录数据包日志(true或false,仅用于调试,开启将产生大量日志文件)
const IdSTime = 1000;
//允许延迟时间(单位ms,至少应为500) 源服务器超过允许延迟时间未返回响应包将断开连接
const whiteList = false, blackList = false, IdTime = 30000;
//白名单与黑名单(true或false) 允许客户端发送id的时间,不允许连接的用户将在该时间后踢出(单位ms,至少应为500) 开启白名单后仅允许白名单连接 开启黑名单后将禁止黑名单连接 同时存在白名单和黑名单的用户将禁止连接

/*
 -minecraft服务器代理工具-
    作者: ndzda0
   最后编辑:2020.10.25
  请勿删除说明与版权信息!
  禁止传播修改后的文件!
Copyright (c) 2020 Ndzda studio
*/

var net = require("net"), fs = require("fs");
function b2h(b)
{
    return (Array.prototype.slice.call(b)).map(el => Number(el).toString(16));
}
function h2b(h)
{
    return Buffer.from(new Uint8Array(h.map(el => parseInt(el, 16))));
}
function s2h(s)
{
    var h = [];
    for (var i = 0, Li = s.length; i < Li; i++)h.push((s.charCodeAt(i)).toString(16));
    return h;
}
function h2s(h, i, siz)
{
    var s = "";
    if (!i) i = 0;
    for (Li = (siz == undefined ? h.length : i + siz); i < Li; i++)s += String.fromCharCode(parseInt(h[i], 16));
    return s;
}
function vInt2int(a, l)
{
    if (typeof (a) != "object") return 0;
    var numRead = 0, result = 0, read = 0;
    for (var i = (l ? l : 0), Li = a.length; i < Li; i++)
    {
        read = parseInt(a[i], 16);
        var value = (read & 0b01111111);
        result |= (value << (7 * numRead));
        numRead++;
        if ((read & 0b10000000) == 0) break;
    }
    return result;
}
function vIntGetB(a, l)
{
    if (typeof (a) != "object") return 0;
    var numRead = 0;
    for (var i = (l ? l : 0), Li = a.length; i < Li; i++)
    {
        numRead++;
        if ((parseInt(a[i], 16) & 0b10000000) == 0) break;
    }
    return numRead;
}
function int2vInt(n)
{
    var h = [];
    do
    {
        var temp = n & 0b01111111;
        n >>>= 7;
        if (n != 0) temp |= 0b10000000;
        h.push(temp.toString(16));
    } while (n != 0);
    return (h.length ? h : ['0']);
}

var log = fs.createWriteStream('./log.txt', { flags: "w", encoding: "utf-8" });
var DIpH = s2h((DisIp ? DisIp : SerIp));

function ModifyPackage(a)
{
    var LenOfIpLen = vIntGetB(a, 3), LenIp = vInt2int(a, 3);
    var OIpZ = h2s(a, 3 + LenOfIpLen, LenIp).indexOf('\0');
    if (OIpZ == -1) OIpZ = LenIp;
    var DSuf = a.slice(3 + LenOfIpLen + OIpZ, 3 + LenOfIpLen + OIpZ + LenIp - OIpZ);
    DSuf.splice(0, 0, ...DIpH);
    DSuf.splice(-1, 0, ...DESuf);
    var NewLen = DSuf.length;
    a.splice(3, LenOfIpLen + LenIp, ...int2vInt(NewLen), ...DSuf);
    a.splice(0, 1, ...int2vInt(vInt2int(a, 0) + (NewLen - LenIp) + (vIntGetB(int2vInt(NewLen)) - LenOfIpLen)));
    return a;
}

var WL = {}, BL = {};
var ser = net.createServer(function (soc)
{
    console.log("[!]client connected");
    soc.on("end", function ()
    {
        console.log("[!]client disconnect");
    });
    var NewC = DVisit, ODPing = false, PlName = "";
    var cli = net.connect({ port: SerPor, host: SerIp }, function ()
    {
        console.log("[!]proxy client initialization completed");
        soc.on("data", function (data)
        {
            var h = b2h(data);
            if (OLog)
            {
                log.write("C> ");
                for (var i = 0, Li = h.length; i < Li; i++)log.write((h[i].length < 2 ? '0' + h[i] : h[i]) + ' ');
                log.write('\n');
            }
            if (NewC)
            {
                if (!PlName)
                {
                    for (var i = 0, Li = h.length, BSiz = 0, NSiz = 0; i < Li; i += BSiz + NSiz)
                    {
                        //console.log("debug|", i);
                        BSiz = vInt2int(h, i);
                        NSiz = vIntGetB(h, i);
                        if (h[i + NSiz] == '0' && vInt2int(h, i + NSiz + 1) == BSiz - 2)
                        {
                            PlName = h2s(h, i + NSiz + 2, vInt2int(h, i + NSiz + 1));
                            console.log("player|", PlName);
                            break;
                        }
                    }
                }
                cli.write(h2b(ModifyPackage(h)));
                //console.log("debug|", ModifyPackage(b2h(data)));
                NewC = false;
                return;
            }
            cli.write(data);
        });
        setTimeout(() =>
        {
            if ((whiteList || blackList) && !PlName)
            {
                cli.destroy();
                soc.destroy();
            }
        }, IdTime);
    });
    cli.on("error", function (err)
    {
        console.log("errS|" + err);
        soc.destroy();
    });
    soc.on("error", function (err)
    {
        console.log("errC|" + err);
        if (!ODPing) cli.destroy();
    });
    cli.on("data", function (data)
    {
        var h = b2h(data);
        if (OLog)
        {
            log.write("S> ");
            for (var i = 0, Li = h.length; i < Li; i++)log.write((h[i].length < 2 ? '0' + h[i] : h[i]) + ' ');
            log.write('\n');
        }
        if (!ODPing) soc.write(data);
    });
    cli.on("end", function ()
    {
        console.log("[!]server disconnect");
    });

});
ser.listen(NPort, function ()
{
    console.log("[!]proxy server initialization completed");
});
if (whiteList)
{
    var WLA = fs.readFileSync("./wl.txt").trim().split('\n');
    for (var i = 0, Li = WLA.length; i < Li; i++)WL[WLA[i]] = 1;
}
if (blackList)
{
    var BLA = fs.readFileSync("./bl.txt").trim().split('\n');
    for (var i = 0, Li = BLA.length; i < Li; i++)BL[BLA[i]] = 1;
}
