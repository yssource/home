+++
title = "Flash 动画 模块详解"
author = ["Jimmy.M.Gong"]
description = "Webpack, Angular.js, Ionic, Wechat, H5, Adobe Edge, Wordpress, Magento, 音乐贺卡"
date = 2017-07-20
tags = ["Webpack", "Angular.js", , , , , , , "Edge", , , ]
categories = ["mobile", ]
draft = false
[menu.foo]
  parent = "main"
  weight = 10
  identifier = "music-greeting"
+++

## [Flash 动画 实现详解](https://yssource.github.io/2016/10/26/audi/#audi_menu_flash) {#flash-动画-实现详解}

{{< highlight sh "linenos=table, linenostart=1" >}}
├── ./flash
│   ├── ./flash/controllers
│   │   ├── ./flash/controllers/flash.controller.js
│   │   ├── ./flash/controllers/flash.controller.test.js
│   │   └── ./flash/controllers/index.js
│   ├── ./flash/index.js
│   ├── ./flash/services
│   │   ├── ./flash/services/flash.service.js
│   │   ├── ./flash/services/flash.service.test.js
│   │   └── ./flash/services/index.js
│   ├── ./flash/tests.webpack.js
│   └── ./flash/views
│       └── ./flash/views/home.html
{{< /highlight >}}

<div class="HTML">
  <div></div>

<!--more-->

</div>

1.  File: flash/index.js[^fn:1]

    <a id="orgafb39a8"></a>
    {{< highlight javascript "linenos=table, linenostart=1" >}}
    'use strict';
    var angular = require('angular');
    var modulename = 'flash';

    module.exports = function(namespace) {
        var fullname = namespace + '.' + modulename;

        var app = angular.module(fullname, ['ui.router', 'ionic'/*, 'ngResource'*/]);
        require('./services')(app);
        require('../utils/services')(app);
        app.namespace = app.namespace || {};

        var configRoutesDeps = ['$stateProvider', '$urlRouterProvider'];
        var configRoutes = function($stateProvider, $urlRouterProvider) {
            //$urlRouterProvider.otherwise('/');
            $stateProvider.state('app.menu.flash', {
                url: '/flash/',
                views: {
                    'menuContent@app.menu': {
                        //template: require('./views/home.html'),
                        templateProvider: ['$q', function ($q) {
                            var deferred = $q.defer();
                            require.ensure(['./views/home.html'], function () {
                                var template = require('./views/home.html');
                                deferred.resolve(template);
                            });
                            return deferred.promise;
                        }],
                        controller: fullname + '.' + 'flashCtrl'
                    }
                },
                resolve: {
                    loadFlashController: ['$q', '$ocLazyLoad', function ($q, $ocLazyLoad) {
                        var deferred = $q.defer();
                        require.ensure([], function () {
                            require('./controllers/index.js')(app);
                            require('./services/index.js')(app);
                            $ocLazyLoad.load({
                                name: app.name
                            });
                            deferred.resolve(/*module.controller*/);
                        });

                        return deferred.promise;
                    }]
                }

            });
        };
        configRoutes.$inject = configRoutesDeps;
        app.config(configRoutes);

        return app;
    };
    {{< /highlight >}}

2.  File: flash/controller/flash.controller.js[^fn:2]

    {{< highlight javascript "linenos=table, linenostart=1" >}}
    'use strict';
    var controllername = 'flashCtrl';

    module.exports = function (app) {
        var fullname = app.name + '.' + controllername;
        /*jshint validthis: true */

        var deps = ['$scope', '$state', 'flashService'];

        function controller($scope, $state, flashService) {
            var vm = this;
            vm.controllername = fullname;

            var activate = function () {
                flashService.createDB().then(function () {
                    flashService.initDB();
                    // Get all flash records from the database.
                    flashService.getAllFlashs().then(function (flashs) {
                        $scope.flashs = flashs;
                    });
                });
            };
            activate();

            $scope.setupflash = function (cat) {
                window.localStorage.fid = cat.id;
                $state.go('app.menu.home', {
                    fid: window.localStorage.fid,
                    mid: window.localStorage.mid,
                    pid: window.localStorage.pid,
                    tid: window.localStorage.tid,
                    did: window.localStorage.did,
                    cid: window.localStorage.cid
                });
            };
        }

        controller.$inject = deps;
        app.controller(fullname, controller);
    };
    {{< /highlight >}}
3.  file: flash/service/flash.service.js[^fn:3]

{{< highlight javascript "linenos=table, linenostart=1" >}}
'use strict';
var servicename = 'flashService';

var PouchDB = require('pouchdb');
var pouchFlash = new PouchDB('flashs', {adapter: 'websql', size: 1});
if (!pouchFlash.adapter) { // websql not supported by this browser
    pouchFlash = new PouchDB('flashs');
}
PouchDB.plugin(require('pouchdb-find'));
if (process.env.SENTRY_MODE !== 'prod') {
    PouchDB.debug.enable('*');
} else {
    PouchDB.debug.disable();
}

module.exports = function (app) {

    var dependencies = ['$q', 'utilsService'];

    function service($q, utilsService) {
        var _db;
        var _flashs = [];
        var _ddoc = [];

        return {
            createDB: createDB,
            initDB: initDB,
            getAllFlashs: getAllFlashs
        };

        function createDB() {
            var deferred = $q.defer();

            function myBulkDocs(_ddoc) {
                pouchFlash.bulkDocs({docs: _ddoc}, function (err, response) {
                    // handle err or response
                    if (err) {
                        console.log(err);
                    } else {
                        window.localStorage.setItem('flashDBCreated', true);
                        deferred.resolve();
                    }
                });
            }

            //window.localStorage.removeItem("flashDBCreated");
            if (!JSON.parse(window.localStorage.getItem('flashDBCreated'))) {
                pouchFlash.destroy().then(function (response) {
                    // success
                    // Creates the database or opens if it already exists
                    pouchFlash = new PouchDB('flashs', {adapter: 'websql', size: 1});
                    if (!pouchFlash.adapter) { // websql not supported by this browser
                        pouchFlash = new PouchDB('flashs');
                    }

                    pouchFlash.createIndex({
                        index: {
                            fields: ['id']
                        }
                    }).then(function (result) {
                        pouchFlash.find({
                            selector: {id: 1},
                            fields: ['id'/*, 'name'*/],
                            sort: ['id']
                        }).then(function (result) {
                            if (!result.docs.length) {
                                if (!_ddoc.length) {
                                    utilsService.getFlashResource().then(function (source) {
                                        _ddoc = source;
                                        myBulkDocs(_ddoc);
                                    });
                                } else {
                                    myBulkDocs(_ddoc);
                                }
                            }
                        }).catch(function (err) {
                            // ouch, an error
                            console.log(err);
                        });
                    }).catch(function (err) {
                        // ouch, an error
                        console.log(err);
                    });
                }).catch(function (err) {
                    console.log(err);
                    alert(err);
                });
            } else {
                deferred.resolve();
            }
            return deferred.promise;
        }

        function initDB() {
            // Creates the database or opens if it already exists
            _db = new PouchDB('flashs', {adapter: 'websql', size: 1});
            if (!_db.adapter) { // websql not supported by this browser
                _db = new PouchDB('flashs');
            }
        }

        function getAllFlashs() {
            if (!_flashs.length) {
                return $q.when(
                    _db.find({
                            selector: {id: {$gt: 0}},
                        sort: ['id']
                        /* skip: 1,
                           limit: 5*/
                    }))
                    .then(function (docs) {
                        _flashs = docs.docs.map(function (row) {
                            return row;
                        });

                        return _flashs;
                    }).catch(function (err) {
                        console.log('err ', err);
                    });
            } else {
                // Return cached data as a promise
                return $q.when(_flashs);
            }
        }
    }

    service.$inject = dependencies;
    app.factory(servicename, service);
};
{{< /highlight >}}

[^fn:1]: flash/index.js
[^fn:2]: flash/controoler/flash.controller.js
[^fn:3]: flash/service/flash.service.js
