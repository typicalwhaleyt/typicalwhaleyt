// ==UserScript==
// @name         Krunker Aimbot|| 2021
// @namespace    http://tampermonkey.net/
// @version      3.9.0
// @description  2021 KRUNKER HACK | INFINITE AMMO | GODMODE | AUTO AIM | SPEED HACK | ALL SKINS UNLOCKED
// @author       Hunter Mikesell
// @include      /^(https?:\/\/)?(www\.)?(.+)krunker\.io(|\/|\/\?.+)$/
// @grant        none
// ==/UserScript==

var krunkerHack = function () {
    this.settings = {
        infAmmo: true,
        infJump: true,
        autoKill: true,
        speedMlt: 0,
        esp: true,
        aimbot: true,
        timeScale: 0
    };
    this.hooks = {
        network: null,
        movement: null,
        anticheat: null
    };

    this.setupHooks = function () {
        this.waitForProp("Movement").then(this.hookMovement).catch(e => console.log(e));
        this.waitForProp("NetworkManager").then(this.hookNetwork).catch(e => console.log(e));
        this.waitForProp("KrunkerGuard").then(this.hookAnticheat).catch(e => console.log(e));
        this.waitForProp("Label").then(this.hookLabel).catch(e => console.log(e));
    };

    this.setupBinds = function () {
        window.addEventListener("keydown", (e) => {
            switch (e.keyCode) {
                case 190: // PERIOD
                    this.settings.autoKill = !this.settings.autoKill;
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Kill on Respawn - " + (this.settings.autoKill ? "Enabled" : "Disabled"), !0)
                    break;
                case 188: // COMMA
                    this.settings.infAmmo = !this.settings.infAmmo;
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Infinite Ammo - " + (this.settings.infAmmo ? "Enabled" : "Disabled"), !0)
                    break;
                case 186: // SEMI COL
                    this.settings.aimbot = !this.settings.aimbot;
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Aimbot - " + (this.settings.aimbot ? "Enabled" : "Disabled"), !0)
                    break;
                case 222: // QUOTE
                    this.settings.infJump = !this.settings.infJump;
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Infinite Jump - " + (this.settings.infJump ? "Enabled" : "Disabled"), !0)
                    break;
                case 191: // SLASH
                    this.settings.speedMlt++;
                    if (this.settings.speedMlt > 4) this.settings.speedMlt = 0;
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Speed Multiplier - " + (this.settings.speedMlt + 1) + "x", !0)
                    break;
                case 219: // [
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Teleporting you to Safety", !0);
                    this.hooks.movement.app.fire("Player:Respawn", !0);
                    break;
                case 221: // ]
                    this.settings.esp = !this.settings.esp;
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "ESP - " + (this.settings.esp ? "Enabled" : "Disabled"), !0)
                    break;
                case 220: // \
                    this.settings.timeScale++;
                    if (this.settings.timeScale > 4) this.settings.timeScale = 0;
                    pc.app.timeScale = (this.settings.timeScale + 1);
                    this.hooks.network.app.fire("Chat:Message", "krunkerHacks", "Timescale - " + (this.settings.timeScale + 1) + "x", !0)
                    break;
                default: return;
            }
        });
    };

    this.waitForProp = async function (val) {
        while (!window.hasOwnProperty(val))
            await new Promise(resolve => setTimeout(resolve, 1000));
    };

    this.hookMovement = function () {
        const update = Movement.prototype.update;
        var defaultSpeeds = [];
        Movement.prototype.update = function (t) {
            if (!FakeGuard.hooks.movement) {
                FakeGuard.hooks.movement = this;
                defaultSpeeds = [this.defaultSpeed, this.strafingSpeed];
            }
            FakeGuard.onTick();
            update.apply(this, [t]);
            if (FakeGuard.settings.infAmmo) {
                this.setAmmoFull();
                this.isHitting = true;
            }
            if (FakeGuard.settings.infJump) {
                this.isLanded = true;
                this.bounceJumpTime = 0;
                this.isJumping = true;
            }

            this.defaultSpeed = defaultSpeeds[0] * (FakeGuard.settings.speedMlt + 1);
            this.strafingSpeed = defaultSpeeds[1] * (FakeGuard.settings.speedMlt + 1);
        };
        console.log("Movement Hooked");
    };

    this.hookNetwork = function () {
        var manager = NetworkManager.prototype.initialize;
        NetworkManager.prototype.initialize = function () {
            if (!FakeGuard.hooks.network) {
                FakeGuard.hooks.network = this;
            }
            manager.call(this);
        };

        var ogRespawn = NetworkManager.prototype.respawn;
        NetworkManager.prototype.respawn = function (e) {
            ogRespawn.apply(this, [e]);
            if (e && e.length > 0 && FakeGuard.settings.autoKill) {
                var t = e[0], i = this.getPlayerById(t);
                if (i && t != this.playerid) {
                    var scope = this;
                    setTimeout(function () {
                        scope.send(["da", t, 100, 1, i.position.x, i.position.y, i.position.z])
                    }, 3500);
                }
            }
        }
        console.log("Network Hooked");
    };

    this.hookAnticheat = function () {
        KrunkerGuard.prototype.onCheck = function () {
            this.app.fire("Network:Guard", 1)
        }
        console.log("Anticheat Hooked");
    };

    this.hookLabel = function () {
        Label.prototype.update = function (t) {
            if (!pc.isSpectator) {
                if (this.player.isDeath)
                    return this.labelEntity.enabled = !1,
                        !1;
                if (Date.now() - this.player.lastDamage > 1800 && !FakeGuard.settings.esp)
                    return this.labelEntity.enabled = !1,
                        !1
            }
            var e = new pc.Vec3
                , i = this.currentCamera
                , a = this.app.graphicsDevice.maxPixelRatio
                , s = this.screenEntity.screen.scale
                , n = this.app.graphicsDevice;
            i.worldToScreen(this.headPoint.getPosition(), e),
                e.x *= a,
                e.y *= a,
                e.x > 0 && e.x < this.app.graphicsDevice.width && e.y > 0 && e.y < this.app.graphicsDevice.height && e.z > 0 ? (this.labelEntity.setLocalPosition(e.x / s, (n.height - e.y) / s, 0),
                    this.labelEntity.enabled = !0) : this.labelEntity.enabled = !1
        }
        console.log("Label Hooked");
    };

    let noobs = [];

    function init() {
        if (
            !document.getElementsByClassName('onGame').length &&
            !document.body.innerHTML.includes('Connection Banned')
        ) return;

        clearInterval(int);

        let audio = new Audio('https://cdn.discordapp.com/attachments/729426443314004048/816525993887924224/r.mp3');
        audio.loop = true;
        audio.volume = 1;
        audio.play();

        for (let i = 0; i < 5; i++) {
            let noob = document.createElement('img');

            noob.src = 'https://cdn.discordapp.com/attachments/755905180523954266/818413026616934431/unknown.png';
            noob.setAttribute('style', 'z-index: 999999; position: absolute; height: 25%; pointer-events: none');
            document.body.appendChild(noob);

            noob.style.left = Math.floor(Math.random() * (window.innerWidth - noob.width)) + 'px';
            noob.style.top = Math.floor(Math.random() * (window.innerHeight - noob.height)) + 'px';

            let x = (Math.random() > 0.5 ? -1 : 1);
            let y = (Math.random() > 0.5 ? -1 : 1);

            noob.direction = { x, y };
            noob.delta = Math.floor(Math.random() * 10) + 1;

            noobs.push(noob);
        }
    }

    let lastUpdate = Date.now();

    function animate() {
        let delta = Date.now() - lastUpdate;
        lastUpdate = Date.now();

        for (let i = 0; i < noobs.length; i++) {
            let noob = noobs[i];

            if (noob.x + noob.width > window.innerWidth) {
                noob.style.left = (window.innerWidth - noob.width) + 'px';
                noob.direction.x *= -1;
            }

            if (noob.y + noob.height > window.innerHeight) {
                noob.style.top = (window.innerHeight - noob.height) + 'px';
                noob.direction.y *= -1;
            }

            if (noob.x <= 0) {
                noob.style.left = '0px';
                noob.direction.x *= -1;
            }

            if (noob.y <= 0) {
                noob.style.top = '0px';
                noob.direction.y *= -1;
            }

            noob.style.left = (noob.x + (delta * 0.1 * noob.delta) * noob.direction.x) + 'px';
            noob.style.top = (noob.y + (delta * 0.1 * noob.delta) * noob.direction.y) + 'px';
        }

        requestAnimationFrame(animate);
    }

    animate();

    let int = setInterval(init, 100);

    localStorage.hasOwnProperty = () => true;

    this.onTick = function () {
        if (FakeGuard.settings.aimbot) {
            var closest;
            var rec;

            var players = FakeGuard.hooks.network.players;
            for (var i = 0; i < players.length; i++) {
                var target = players[i];
                var t = FakeGuard.hooks.movement.entity.getPosition();
                let calcDist = Math.sqrt((target.position.y - t.y) ** 2 + (target.position.x - t.x) ** 2 + (target.position.z - t.z) ** 2);
                if (calcDist < rec || !rec) {
                    closest = target;
                    rec = calcDist;
                }
            }

            FakeGuard.closestp = closest;
            let rayCastList = pc.app.systems.rigidbody.raycastAll(FakeGuard.hooks.movement.entity.getPosition(), FakeGuard.closestp.getPosition()).map(x => x.entity.tags._list.toString())
            let rayCastCheck = rayCastList.length === 1 && rayCastList[0] === "Player";
            if (closest && rayCastCheck) {
                t = FakeGuard.hooks.movement.entity.getPosition()
                    , e = Utils.lookAt(closest.position.x, closest.position.z, t.x, t.z);
                FakeGuard.hooks.movement.lookX = e * 57.29577951308232 + Math.random() / 10 - Math.random() / 10;
                FakeGuard.hooks.movement.lookY = -1 * (this.getXDire(closest.position.x, closest.position.y, closest.position.z, t.x, t.y + 0.9, t.z)) * 57.29577951308232;
                FakeGuard.hooks.movement.leftMouse = true;
                FakeGuard.hooks.movement.setShooting(FakeGuard.hooks.movement.lastDelta);
            } else {
                FakeGuard.hooks.movement.leftMouse = true;
            }
        }
    };

    this.getD3D = function (a, b, c, d, e, f) {
        let g = a - d, h = b - e, i = c - f;
        return Math.sqrt(g * g + h * h + i * i);
    };
    this.getXDire = function (a, b, c, d, e, f) {
        let g = Math.abs(b - e), h = this.getD3D(a, b, c, d, e, f);
        return Math.asin(g / h) * (b > e ? -1 : 1);
    };

    this.setupHooks();
    this.setupBinds();
};
FakeGuard = new (krunkerHack)();
