<template>
    <div id="playerWrap" class="player-wrap" :style="`height: ${wrapSize.height}px; width: ${wrapSize.width}px;`">
        <!-- 占位模板 -->
        <div class="player-placeholder">
            <div class="player-placeholder-top"></div>
            <div class="player-placeholder-bottom">
                <div class="player-placeholder-bottom-left"></div>
                <div class="player-placeholder-bottom-right"></div>
            </div>
        </div>
        <!-- 主体 -->
        <div class="player">
            <div class="player-primary-area">
                <!-- 视频播放区域 -->
                <div class="player-video-area" id="video-area" :class="{ 'state-paused': pause, 'state-buff': buff && !pause }" :data-screen="screenType" :data-ctrl-hidden="ctrlHidden" @mouseleave="ctrlHidden = true">
                    <!-- 视频 -->
                    <div class="player-video-perch">
                        <div class="player-video-wrap">
                            <video ref="videoPlayer" :src="videoUrl"
                                @loadedmetadata="videoCanPlay"
                                @timeupdate="timeUpdate"
                                @progress="updateBufferingBar"
                                @waiting="buff = true"
                                @canplay="buff = false"
                                @ended="ended"
                                :loop="setting.loop"
                            ></video>
                        </div>
                    </div>
                    <!-- 状态 -->
                    <div class="player-state">
                        <div class="player-state-play"></div>
                        <div class="player-state-buff-icon">
                            <LoadingBuff></LoadingBuff>
                        </div>
                        <div class="player-state-buff-text">
                            <span>正在缓冲...</span><span>{{ bufferSpeed }}KB/s</span>
                        </div>
                    </div>
                    <!-- 加载 -->
                    <div class="player-loading-panel" :style="canPlay ? 'display: none;' : 'display: block;'">
                        <PlayerLoading></PlayerLoading>
                    </div>
                    <!-- 蒙版 -->
                    <div class="player-masking" @mousemove="showCtrl" @click="handleClick" @dblclick="handleDblClick"></div>
                    <!-- 顶部 -->
                    <div class="player-top-wrap">
                        <div class="player-top-left">
                            <div class="player-title">{{ title }}</div>
                            <div v-if="user.uid">
                                <div class="player-follow" @mouseenter="showCtrlAlweys">
                                    <img class="player-follow-face" :src="user.avatar_url" alt="">
                                    <svg t="1700578364917" class="icon player-follow-icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3687" width="12" height="12"><path d="M512 972.8a51.2 51.2 0 0 1-51.2-51.2V102.4a51.2 51.2 0 0 1 102.4 0v819.2a51.2 51.2 0 0 1-51.2 51.2z" p-id="3688" fill="#FFFFFF"></path><path d="M921.6 512H102.4" fill="#FFFFFF" p-id="3689"></path><path d="M921.6 563.2H102.4a51.2 51.2 0 0 1 0-102.4h819.2a51.2 51.2 0 0 1 0 102.4z" p-id="3690" fill="#FFFFFF"></path></svg>
                                    <span class="player-follow-text">关注</span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <!-- 控制器 -->
                    <div class="player-control-wrap" @mouseenter="showCtrlAlweys">
                        <div class="player-control-mask"></div>
                        <div class="player-control-entity">
                            <div class="player-control-top">
                                <PlayerProgress :currentPer="currentPer" :bufferPer="bufferPer" @changeCurrent="changeCurrentPer"></PlayerProgress>
                            </div>
                            <div class="player-control-bottom">
                                <div class="player-control-bottom-left">
                                    <div class="player-ctrl-btn player-ctrl-prev" v-if="false">
                                        <!-- 上一个 -->
                                        <div class="player-ctrl-btn-icon">
                                            <span class="common-svg-icon">
                                                <svg t="1700643760528" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3961" width="22" height="22"><path d="M343.95259221 459.21031851L712.1969664 178.16268117c43.70934215-33.36821646 106.7137627-2.19481771 106.71376157 52.78865294v562.09527581c0 54.98347065-63.00442055 86.15892423-106.71376157 52.79070891L343.95259221 564.78968149c-34.81704221-26.57620082-34.81704221-79.00316103 0-105.57936298zM201.32437789 817.83284281V443.37392867v-187.23562268c0-22.33247517-6.71803278-68.8079451 19.40810752-86.29044906 29.04023267-19.42043762 67.97975097 1.37689885 67.97975096 36.31724544v561.69453681c0 22.33247517 6.73036288 68.80589027-19.39577628 86.29044907-29.05256391 19.42043762-67.99208221-1.37895367-67.9920822-36.31724544z" fill="#FFFFFF" p-id="3962"></path></svg>
                                            </span>
                                        </div>
                                    </div>
                                    <div class="player-ctrl-btn player-ctrl-play">
                                        <!-- 播放 -->
                                        <div class="player-ctrl-btn-icon player-ctrl-play-play" :style="pause ? '' : 'display: none;'" @click="playVideo">
                                            <span class="common-svg-icon">
                                                <svg t="1700644047374" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4206" width="22" height="22"><path d="M847.52000001 429.80266667L344.50133334 101.152c-78.176-45.184-142.21866667-8.18133333-142.21866667 82.13333333v657.22666667c0 90.368 64.04266667 127.30666667 142.21866667 82.19733333L847.52000001 594.13333333c78.25066667-45.25866667 78.25066667-119.14666667 0-164.33066666z m0 0" fill="#FFFFFF" p-id="4207"></path></svg>
                                            </span>
                                        </div>
                                        <!-- 暂停 -->
                                        <div class="player-ctrl-btn-icon player-ctrl-play-pause" :style="!pause ? '' : 'display: none;'" @click="pauseVideo">
                                            <span class="common-svg-icon">
                                                <svg t="1700644104462" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4447" width="22" height="22"><path d="M735.9 64.9c-51.2 0-96 44.7-96 95.8v702.6c0 51.1 44.8 95.8 96 95.8s96-44.7 96-95.8V160.7c0-51.1-44.8-95.8-96-95.8z m-447.8 0c-51.2 0-96 44.7-96 95.8v702.6c0 51.1 44.8 95.8 96 95.8s96-44.7 96-95.8V160.7c-0.1-51.1-44.9-95.8-96-95.8z" p-id="4448" fill="#FFFFFF"></path></svg>
                                            </span>
                                        </div>
                                    </div>
                                    <div class="player-ctrl-btn player-ctrl-next" v-if="false">
                                        <!-- 下一个 -->
                                        <div class="player-ctrl-btn-icon">
                                            <span class="common-svg-icon">
                                                <svg t="1700644237259" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4688" width="22" height="22"><path d="M680.0474078 564.78968149L311.8030336 845.83731883c-43.70934215 33.36821646-106.7137627 2.19481771-106.71376157-52.78865295l0-562.0952758c0-54.98347065 63.00442055-86.15892423 106.71376157-52.79070891L680.0474078 459.21031851c34.81704221 26.57620082 34.81704221 79.00316103 0 105.57936298zM822.67562211 206.16715719L822.67562211 580.62607133l0 187.23562268c0 22.33247517 6.71803278 68.8079451-19.40810752 86.29044906-29.04023267 19.42043762-67.97975097-1.37689886-67.97975096-36.31724544l0-561.69453681c0-22.33247517-6.73036288-68.80589027 19.39577628-86.29044907 29.05256391-19.42043761 67.99208221 1.37895367 67.9920822 36.31724544z" fill="#FFFFFF" p-id="4689"></path></svg>
                                            </span>
                                        </div>
                                    </div>
                                    <div class="player-ctrl-btn player-ctrl-time">
                                        <div class="player-ctrl-time-label">
                                            <span class="player-ctrl-time-current">{{ handleTime(currentTime) }}</span>
                                            <span class="player-ctrl-time-divide">/</span>
                                            <span class="player-ctrl-time-duration">{{ handleTime(duration) }}</span>
                                        </div>
                                    </div>
                                </div>

                                <div class="player-control-bottom-right">
                                    <!-- 清晰度 -->
                                    <div class="player-ctrl-btn player-ctrl-quality" v-if="false"
                                        @mouseenter="enterBtn('player-ctrl-quality', 2)"
                                        @mouseleave="leaveBtn('player-ctrl-quality', 2)"
                                    >
                                        <div class="player-ctrl-quality-result">{{ '720P 高清' }}</div>
                                    </div>
                                    <!-- 选集 -->
                                    <div class="player-ctrl-btn player-ctrl-eplist" v-if="false"
                                        @mouseenter="enterBtn('player-ctrl-eplist', 3)"
                                        @mouseleave="leaveBtn('player-ctrl-eplist', 3)"
                                    >
                                        <div class="player-ctrl-eplist-result">选集</div>
                                    </div>
                                    <!-- 倍数 -->
                                    <div class="player-ctrl-btn player-ctrl-playbackrate"
                                        @mouseenter="enterBtn('player-ctrl-playbackrate', 4)"
                                        @mouseleave="leaveBtn('player-ctrl-playbackrate', 4)"
                                    >
                                        <div class="player-ctrl-playbackrate-result">{{ playbackrate }}</div>
                                        <ul class="player-ctrl-playbackrate-menu">
                                            <li class="player-ctrl-playbackrate-menu-item" :class="{'state-active': playbackrate === '2.0x'}" @click="changePlaybackRate(2.0)">2.0x</li>
                                            <li class="player-ctrl-playbackrate-menu-item" :class="{'state-active': playbackrate === '1.5x'}" @click="changePlaybackRate(1.5)">1.5x</li>
                                            <li class="player-ctrl-playbackrate-menu-item" :class="{'state-active': playbackrate === '1.25x'}" @click="changePlaybackRate(1.25)">1.25x</li>
                                            <li class="player-ctrl-playbackrate-menu-item" :class="{'state-active': playbackrate === '倍速'}" @click="changePlaybackRate(1.0)">1.0x</li>
                                            <li class="player-ctrl-playbackrate-menu-item" :class="{'state-active': playbackrate === '0.75x'}" @click="changePlaybackRate(0.75)">0.75x</li>
                                            <li class="player-ctrl-playbackrate-menu-item" :class="{'state-active': playbackrate === '0.5x'}" @click="changePlaybackRate(0.5)">0.5x</li>
                                        </ul>
                                    </div>
                                    <!-- 设置 -->
                                    <div class="player-ctrl-btn player-ctrl-setting"
                                        @mouseenter="enterBtn('player-ctrl-setting', 5)"
                                        @mouseleave="leaveBtn('player-ctrl-setting', 5)"
                                    >
                                        <div class="player-ctrl-btn-icon">
                                            <span class="common-svg-icon">
                                                <svg t="1700669811042" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="7338" width="22" height="22"><path d="M882.34150875 504.54345955c0-49.57218555 27.61681646-93.75909188 71.66563872-110.19109768-22.09345317-77.18900202-66.14227541-148.85464074-121.23782427-203.95018957-38.52545897 27.61681646-93.62100781 33.00209567-137.80791415 11.0467266-44.04882225-27.61681646-66.14227541-71.66563872-60.61891213-121.23782427-38.52545897-11.04672658-82.7123653-16.57008987-121.23782427-16.57008988-44.04882225 0-82.7123653 5.5233633-121.23782426 16.57008988 5.5233633 44.04882225-16.57008987 93.62100781-60.61891212 121.23782427s-99.14437109 22.09345317-137.80791414-11.0467266c-55.09554883 55.09554883-99.14437109 126.76118755-121.23782426 203.95018957 44.04882225 16.57008987 71.66563872 60.61891214 71.66563872 110.19109768s-27.61681646 93.62100781-71.66563872 110.19109769c22.09345317 77.18900202 66.14227541 148.85464074 121.23782426 203.95018957 38.52545897-27.61681646 93.62100781-33.00209567 137.80791414-11.0467266 44.04882225 27.61681646 66.14227541 71.66563872 60.61891212 121.23782426 38.52545897 11.04672658 82.7123653 16.57008987 121.23782426 16.5700899 44.04882225 0 82.7123653-5.5233633 121.23782427-16.5700899-5.5233633-44.04882225 16.57008987-93.62100781 60.61891213-121.23782426s99.14437109-22.09345317 137.80791415 11.0467266c55.09554883-55.09554883 99.14437109-126.76118755 121.23782427-203.95018957-44.18690634-16.43200579-71.66563872-60.61891214-71.66563872-110.19109769zM513.10467265 686.40019595c-104.66773438 0-187.3800997-82.7123653-187.38009967-181.8567364 0-99.14437109 82.7123653-181.85673639 187.38009967-181.85673638 104.66773438 0 187.3800997 82.7123653 187.3800997 181.85673638-0.13808409 99.28245518-82.7123653 181.85673639-187.3800997 181.8567364z m0-71.66563871c60.8950803 0 110.19109769-49.29601739 110.19109769-110.19109769 0-60.8950803-49.29601739-110.19109769-110.19109769-110.19109768-60.8950803 0-110.19109769 49.29601739-110.19109768 110.19109768-0.13808409 60.8950803 49.29601739 110.19109769 110.19109768 110.19109769z m0 0" fill="#FFFFFF" p-id="7339"></path></svg>
                                            </span>
                                        </div>
                                        <div class="player-ctrl-setting-box">
                                            <div class="player-ctrl-setting-menu v-ui v-ui-panel v-ui-dark">
                                                <div class="v-ui-area">
                                                    <div class="v-ui-panel-wrap" style="width: 132px; height: 108px;">
                                                        <div class="v-ui-panel-move" style="width: 418px;">
                                                            <div class="v-ui-panel-item v-ui-panel-item-active" style="width: 132px; height: 108px;">
                                                                <div class="player-ctrl-setting-menu-left">
                                                                    <div class="player-ctrl-setting-loop v-ui v-ui-switch" :class="{'checked': setting.loop}" @click="changeSetting(1)">
                                                                        <div class="v-ui-area">
                                                                            <div class="v-ui-switch-label">
                                                                                <span class="v-ui-switch-name">洗脑循环</span>
                                                                                <span class="v-ui-switch-body">
                                                                                    <span class="v-ui-switch-dot"><span></span></span>
                                                                                </span>
                                                                            </div>
                                                                        </div>
                                                                    </div>
                                                                    <div class="player-ctrl-setting-autoplay v-ui v-ui-switch" :class="{'checked': setting.autoplay}" @click="changeSetting(2)">
                                                                        <div class="v-ui-area">
                                                                            <div class="v-ui-switch-label">
                                                                                <span class="v-ui-switch-name">自动开播</span>
                                                                                <span class="v-ui-switch-body">
                                                                                    <span class="v-ui-switch-dot"><span></span></span>
                                                                                </span>
                                                                            </div>
                                                                        </div>
                                                                    </div>
                                                                    <div class="player-ctrl-setting-autonext v-ui v-ui-switch" :class="{'checked': setting.autonext}" @click="changeSetting(3)">
                                                                        <div class="v-ui-area">
                                                                            <div class="v-ui-switch-label">
                                                                                <span class="v-ui-switch-name">自动切集</span>
                                                                                <span class="v-ui-switch-body">
                                                                                    <span class="v-ui-switch-dot"><span></span></span>
                                                                                </span>
                                                                            </div>
                                                                        </div>
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <!-- 音量 -->
                                    <div class="player-ctrl-btn player-ctrl-volume"
                                        @mouseenter="enterBtn('player-ctrl-volume', 6)"
                                        @mouseleave="leaveBtn('player-ctrl-volume', 6)"
                                    >
                                        <div class="player-ctrl-btn-icon" @click="changeMutedState">
                                            <span class="common-svg-icon">
                                                <svg v-if="isMuted" t="1700670251969" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4236" width="22" height="22"><path d="M225.680461 326.598406c-0.419556-0.019443-0.818645-0.019443-1.237177-0.019443L101.812315 326.578963c-22.753213 0-40.876989 18.24248-40.876989 40.777729l0 286.336424c0 22.534226 18.302855 40.777729 40.876989 40.777729l122.629945 0c0.079818 0 0.119727 0 0.198521 0l0 0.157589 300.289204 194.444551c7.125281 6.108115 16.405645 9.781784 26.526143 9.781784 22.573111 0 40.874943-18.301831 40.874943-40.878013 0-1.87572-0.119727-3.711532-0.360204-5.509481L591.970868 168.58151c0.239454-1.795902 0.360204-3.632737 0.360204-5.509481 0-22.574135-18.302855-40.876989-40.874943-40.876989-9.301853 0-17.884322 3.113921-24.750707 8.343015L225.680461 326.598406zM859.567485 510.524392l91.952248-91.951225c11.495822-11.517311 11.576663-30.558993-0.13917-42.274826-11.795651-11.795651-30.636764-11.755742-42.273802-0.140193l-91.953272 91.953272-91.950202-91.953272c-11.639085-11.616572-30.479175-11.655458-42.275849 0.140193-11.715833 11.715833-11.633968 30.757514-0.13917 42.274826l91.952248 91.951225-91.952248 91.953272c-11.494799 11.515265-11.576663 30.556946 0.13917 42.272779 11.796674 11.796674 30.636764 11.756765 42.275849 0.140193l91.950202-91.951225 91.953272 91.951225c11.636015 11.617595 30.477129 11.657504 42.273802-0.140193 11.715833-11.714809 11.634991-30.757514 0.13917-42.272779L859.567485 510.524392z" fill="#FFFFFF" p-id="4237"></path></svg>
                                                <svg v-else t="1700670170727" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3989" width="22" height="22"><path d="M218.495828 334.609871c-0.390903-0.019443-0.773619-0.019443-1.164522-0.019443L100.022553 334.590428c-21.744233 0-39.087227 17.448394-39.087227 39.001269l0 273.866415c0 21.551852 17.505699 38.999223 39.087227 38.999223l117.308753 0c0.057305 0 0.113587 0 0.171915 0l0 0.153496 287.22056 185.975668c6.824429 5.842055 15.691377 9.354042 25.370831 9.354042 21.590737 0 39.096437-17.505699 39.096437-39.095413 0-1.794879-0.124843-3.551896-0.354064-5.270027L568.836985 183.473685c0.229221-1.718131 0.354064-3.475148 0.354064-5.269004 0-21.590737-17.505699-39.096437-39.096437-39.096437-8.895601 0-17.105586 2.977821-23.682375 7.979742L218.495828 334.609871z" fill="#FFFFFF" p-id="3990"></path><path d="M757.858012 953.491133l0.085958 0.075725c123.876332-107.514689 202.211445-266.13329 202.211445-443.041442 0-177.214121-78.603219-336.062965-202.851011-443.61654l-0.11461 0.11461c-4.963035-3.817955-11.17655-6.109138-17.925255-6.109138-16.197914 0-29.322839 13.133112-29.322839 29.321816 0 6.757914 2.28095 12.981662 6.109138 17.926278l-0.333598 0.342808c0.821715 0.706081 1.641383 1.393743 2.462075 2.119267 1.173732 1.202385 2.452865 2.329045 3.817955 3.321652 110.054535 96.710622 179.51349 238.550071 179.51349 396.578224 0 158.02713-69.458955 299.866578-179.51349 396.577201-1.36509 0.99363-2.644223 2.118244-3.817955 3.321652-0.600681 0.533143-1.212618 1.048889-1.822508 1.564635l0.229221 0.230244c-4.152577 5.058203-6.643304 11.530614-6.643304 18.593474 0 16.188704 13.124925 29.321816 29.322839 29.321816C746.317165 960.134437 752.798786 957.651896 757.858012 953.491133z" fill="#FFFFFF" p-id="3991"></path><path d="M713.998085 729.35433l0.23843 0.24764c55.380308-56.43022 89.532129-133.76454 89.532129-219.077577 0-85.409229-34.228569-162.800853-89.704045-219.249493l-0.268106 0.267083c-4.886287-3.64604-10.966773-5.821589-17.543561-5.821589-16.197914 0-29.322839 13.133112-29.322839 29.321816 0 6.566556 2.166339 12.657274 5.822612 17.544585l-0.162706 0.170892c0.773619 0.782829 1.547239 1.584078 2.310625 2.38635 0.075725 0.076748 0.152473 0.171915 0.23843 0.248663 43.3626 45.587268 69.983911 107.248629 69.983911 175.132716 0 67.884087-26.621311 129.544425-69.983911 175.131693-0.085958 0.077771-0.162706 0.171915-0.23843 0.24764-0.706081 0.74599-1.422396 1.471514-2.13871 2.214435l0.144286 0.134053c-3.751441 4.926196-5.976108 11.092639-5.976108 17.754363 0 16.188704 13.124925 29.321816 29.322839 29.321816C702.925912 735.328391 709.072913 733.113957 713.998085 729.35433z" fill="#FFFFFF" p-id="3992"></path></svg>
                                            </span>
                                        </div>
                                        <div class="player-ctrl-volume-box">
                                            <div class="player-ctrl-volume-number">{{ volume }}</div>
                                            <SliderColumn class="player-ctrl-volume-progress" :currentPer="volume / 100" :sliderHeight="60" v-model:order="volumeOrder" @changeCurrent="changeVolume"></SliderColumn>
                                        </div>
                                    </div>
                                    <!-- 全屏 -->
                                    <div class="player-ctrl-btn player-ctrl-full">
                                        <!-- 这里想挂载到#video-area上，但一直挂不上不知道为什么 -->
                                        <el-tooltip :visible="fullTips" placement="top-end" :show-arrow="false">
                                            <template #content>
                                                <span v-if="screenType === 'normal'">进入全屏 (f)</span>
                                                <span v-if="screenType === 'full'">退出全屏 (f)</span>
                                            </template>
                                            <div class="player-ctrl-btn-icon" @click="changeFullScreen" @mouseenter="fullTips = true" @mouseleave="fullTips = false">
                                                <span class="common-svg-icon">
                                                    <svg v-if="screenType === 'normal'" t="1700682460725" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4481" width="22" height="22"><path d="M992 392h-79.99999969V231.99999969h-160.00000031V152h240v240z m-240 480v-79.99999969h160.00000031v-160.00000031h79.99999969v240h-240zM32 632h79.99999969v160.00000031h160.00000031v79.99999969H32v-240zM272 152v79.99999969H111.99999969v160.00000031H32V152h240z m559.99999969 160.00000031H192.00000031v399.99999938h639.99999938V312.00000031z" fill="#FFFFFF" p-id="4482"></path></svg>
                                                    <svg v-if="screenType === 'full'" t="1700682742461" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4722" width="22" height="22"><path d="M896 128H128a64.07 64.07 0 0 0-64 64v640a64.07 64.07 0 0 0 64 64h768a64.07 64.07 0 0 0 64-64V192a64.07 64.07 0 0 0-64-64zM448 672a32 32 0 0 1-64 0v-50.75L246.63 758.63a32 32 0 0 1-45.26-45.26L338.75 576H288a32 32 0 0 1 0-64h128a31.83 31.83 0 0 1 22.35 9.1l0.43 0.43A31.83 31.83 0 0 1 448 544z m374.63-361.37L685.25 448H736a32 32 0 0 1 0 64H608a31.83 31.83 0 0 1-22.35-9.1c-0.14-0.14-0.29-0.28-0.43-0.43A31.83 31.83 0 0 1 576 480V352a32 32 0 0 1 64 0v50.75l137.37-137.38a32 32 0 0 1 45.26 45.26z" fill="#FFFFFF" p-id="4723"></path></svg>
                                                </span>
                                            </div>
                                        </el-tooltip>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <!-- 隐蔽进度条 -->
                    <div class="player-shadow-progress-area">
                        <div class="player-shadow-progress-schedule-wrap">
                            <div class="player-progress-schedule">
                                <div class="player-progress-schedule-buffer" :style="`transform: scaleX(${bufferPer});`"></div>
                                <div class="player-progress-schedule-current" :style="`transform: scaleX(${currentPer});`"></div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- 发送弹幕区域 -->
                <div class="player-sending-area">
                    <div class="player-video-info">
                        <div class="player-video-info-text">
                            {{ population }} 人正在观看
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
import PlayerLoading from './PlayerLoading.vue';
import LoadingBuff from './LoadingBuff.vue';
import PlayerProgress from './PlayerProgress.vue';
import SliderColumn from '@/components/slider/SliderColumn.vue';
// import SliderRow from '@/components/slider/SliderRow.vue';
// import ColorPicker from '@/components/color/ColorPicker.vue';
import { handleTime } from '@/utils/utils.js';
// import { ElMessage } from 'element-plus';

let hideCtrlTimer;  // 隐藏控制器的计时器
let clickTimer; // 单击播放/暂停事件的防抖计时器
let menuShowTimer = [null, null, null, null, null, null, null]; // 每个按钮的提示、菜单等显示延时计时器
let menuHideTimer = [null, null, null, null, null, null, null]; // 每个按钮的提示、菜单等隐藏延时计时器
let speedInterval;  // 更新缓冲网速的定时任务

export default {
    name: "PlayerWrapper",
    components: {
        PlayerLoading,
        LoadingBuff,
        SliderColumn,
        // SliderRow,
        PlayerProgress,
        // ColorPicker,
    },
    data() {
        return {
            isMounted: false,   // 做个三秒挂载延迟，防止increasePlay函数重复触发
            wrapSize: {
                width: 704,
                height: 442,
            },
            videoWidth: 704,
            videoHeight: 396,
            danmuOpen: true,    // 是否开启弹幕
            dmOpacityOrder: false,  // 更新弹幕不透明度slider的命令
            dmList: [],     // 弹幕列表
            lastTimePoint: 0,   // 上一个播放时间点，用于筛选将展示的弹幕
            dmIndex: -1,     // 当前时间点之前的最后一个弹幕的指针，播放更新的时候这个弹幕及之前的都不会展示（使用索引可以提升筛选效率，前提是根据时间点排序的弹幕列表）
            dmSetting: {
                opacity: 100,     // 弹幕透明度
                dmSpeed: 2,   // 弹幕滚动速度，1慢速75px/s 2正常150px/s 4快速300px/s
            },
            rollRow: new Array(12).fill(-1),    // 滚动模式下用于记录每一行上一个弹幕的全呈现时间
            topRow: new Array(12).fill(-1),    // 顶部模式下用于记录每一行上一个弹幕的结束时间
            bottomRow: new Array(12).fill(-1),    // 底部模式下用于记录每一行上一个弹幕的结束时间
            input: '',  // 要发送的弹幕内容
            canPlay: false, // 视频可以播放了
            screenType: 'normal',   // 屏幕类型，normal 正常模式 web 网页全屏 full 屏幕全屏
            ctrlHidden: true,   // 控制器是否隐藏
            pause: false,   // 视频是否暂停中
            buff: false,    // 视频是否正在缓冲
            firstChangeScreen: true,
            currentPer: 0,  // 进度条百分比 0~1
            currentTime: 0, // 当前播放时间
            bufferPer: 0,   // 缓冲百分比 0~1
            bufferSpeed: 0.00, // 缓冲网速 KB/s
            videoSize: 0,   // 视频字节大小
            lastByte: 0,    // 上次缓冲的字节
            isMuted: false, // 是否静音
            volumeOrder: false, // 更新音量slider的命令
            volume: 35,     // 音量
            showDmInput: false, // 是否显示全屏状态下的弹幕输入框
            playbackrate: '倍速',   // 倍速文本
            dmTips: false,  // 弹幕按钮快捷提示是否显示
            fullTips: false,  // 全屏按钮快捷提示是否显示
            setting: {
                loop: false,  // 是否开启洗脑循环
                autoplay: true, // 是否自动播放
                autonext: false,  // 是否自动连播
            },
            dmStyle: {
                fontsize: 25,   // 18 小号 25 标准
                mode: 1,    // 1 滚动 2 顶部 3底部
                color: '#FFFFFF',   // 字体颜色，6位16进制标准格式
            }
        }
    },
    props: {
        // 视频的播放地址
        videoUrl: {
            type: String,
            default() {
                return '';
            }
        },
        // 视频标题
        title: {
            type: String,
            default() {
                return '';
            }
        },
        // 视频总时长
        duration: {
            type: Number,
            default() {
                return 0;
            }
        },
        // UP主资料
        user: {
            type: Object,
            default() {
                return {};
            }
        },
        // 当前观看人数
        population: {
            type: Number,
            default() {
                return 0;
            }
        },
        // 外部时间点跳转并播放指令，大于等于0即触发
        jumpTimePoint: {
            type: Number,
            default() {
                return -1;
            }
        },
        // 外部控制的自动连播
        autonext: {
            type: Boolean,
            default() {
                return false;
            }
        },
    },
    methods: {
        //////// 请求 /////////
        // 增加一个播放量
        async increasePlay() {
            const formData = new FormData();
            formData.append("vid", Number(this.$route.params.vid));
            if (this.$store.state.user.uid) {
                // 如果用户登录了，就算该用户观看了视频
                const res = await this.$post("/favorite/video/play/user", formData, {
                    headers: { Authorization: "Bearer " + localStorage.getItem("teri_token") }
                });
                if (!res.data.data) return;
                const data = res.data.data;
                const atv = {
                    love: data.love === 1 ? true : false,
                    unlove: data.unlove === 1 ? true : false,
                    coin: data.coin,
                    collect: data.collect === 1 ? true : false
                };
                this.$store.commit("updateAttitudeToVideo", atv);
            } else {
                // 否则算游客观看
                await this.$post("/video/play/visitor", formData);
            }
        },


        //////// 事件 /////////
        // 根据窗口大小改变播放器的宽高
        changeWindowSize() {
            const windowWidth = window.innerWidth;
            const windowHeight = window.innerHeight;
            let rightWidth, bottomHeight;   // 整体布局的right-container宽度和底部的发送弹幕栏的高度
            if (windowWidth < 1681) {
                rightWidth = 380;
                bottomHeight = 46;
            } else {
                rightWidth = 441;
                bottomHeight = 56;
            }

            let heigth = (windowHeight - 64) * 0.7;    // 初始使高为屏高的70%
            let width = (heigth - bottomHeight) * (16/9);    // 初始根据比例计算宽

            // 限制宽的范围
            if ((windowWidth - width - rightWidth) < 112) {
                // 表示宽度溢出了，重新计算宽并用其计算高
                width = windowWidth - rightWidth - 112;
                heigth = width * (9/16) + bottomHeight;
            }

            // 最后限制高的范围，并以最后高为准计算最后宽
            if (windowWidth < 1681) {
                heigth = Math.max(424, heigth);
            } else {
                heigth = Math.max(434, heigth);
            }
            heigth = Math.min(1010, heigth);
            width = (heigth - bottomHeight) * (16/9);

            // 更新宽高
            this.wrapSize.width = width;
            this.wrapSize.height = heigth;
            this.videoWidth = width;
            this.videoHeight = heigth - bottomHeight;
            this.$emit('resize', this.wrapSize);
        },

        // 监听键盘按键触发对应事件
        handleKeyboard(event) {
            // console.log(event);
            // 检查当前焦点元素
            var activeElement = document.activeElement;
            var isInputField = activeElement.tagName == 'INPUT' || activeElement.tagName == 'TEXTAREA';
            //  空格键 播放
            if (event.keyCode === 32 && !isInputField) {
                // 阻止默认的空格键行为（滚动页面）
                event.preventDefault();
                this.changePlayState();
            }
            // F键 全屏
            else if (event.keyCode === 70 && !isInputField) {
                this.changeFullScreen();
            }
            // D键 弹幕
            // else if (event.keyCode === 68 && !isInputField) {
            //     this.changeDanmu();
            // }
            // M键 静音
            else if (event.keyCode === 77 && !isInputField) {
                this.changeMutedState();
            }
            // →键 快进5秒
            else if (event.keyCode === 39 && !isInputField) {
                // 阻止默认的→键行为（水平滚动页面）
                event.preventDefault();
                this.$refs.videoPlayer.currentTime += 5;
                // 初始化弹幕索引
                // this.removeAllDanmu();
                this.lastTimePoint = this.$refs.videoPlayer.currentTime;
                this.dmIndex = -1;
                this.rollRow = new Array(12).fill(-1);
                this.topRow = new Array(12).fill(-1);
                this.bottomRow = new Array(12).fill(-1);
            }
            // ←键 回退5秒
            else if (event.keyCode === 37 && !isInputField) {
                // 阻止默认的←键行为（水平滚动页面）
                event.preventDefault();
                this.$refs.videoPlayer.currentTime -= 5;
                // 初始化弹幕索引
                // this.removeAllDanmu();
                this.lastTimePoint = this.$refs.videoPlayer.currentTime;
                this.dmIndex = -1;
                this.rollRow = new Array(12).fill(-1);
                this.topRow = new Array(12).fill(-1);
                this.bottomRow = new Array(12).fill(-1);
            }
            // ↑键 音量+10
            else if (event.keyCode === 38 && !isInputField) {
                // 阻止默认的↑键行为（滚动页面）
                event.preventDefault();
                // console.log("this.volume的值: ", this.volume);
                let volume = (this.volume + 10) / 100;
                // console.log("volume的值: ", volume);
                volume = Math.min(1, volume);
                volume = Math.max(0, volume);
                this.changeVolume(volume);
                this.volumeOrder = true;
            }
            // ↓键 音量-10
            else if (event.keyCode === 40 && !isInputField) {
                // 阻止默认的↓键行为（滚动页面）
                event.preventDefault();
                let volume = (this.volume - 10) / 100;
                volume = Math.min(1, volume);
                volume = Math.max(0, volume);
                this.changeVolume(volume);
                this.volumeOrder = true;
            }
        },

        // 视频已经加载好了可以播放
        async videoCanPlay() {
            this.canPlay = true;
            const durationInSeconds = this.$refs.videoPlayer.duration;
            // 使用720p视频的估计平均比特率（假设 1Mbps）
            const averageBitrate720p = 1000000;
            // 计算估计的视频大小（单位：字节）
            this.videoSize = durationInSeconds * averageBitrate720p / 8;
            if (!this.pause) {
                try {
                    await this.$refs.videoPlayer.play();
                    this.pause = false;
                } catch (err) {
                    this.pause = true;
                }
            }
        },

        // 显示控制器
        showCtrl() {
            clearTimeout(hideCtrlTimer);
            this.ctrlHidden = false;
            hideCtrlTimer = setTimeout(() => {
                this.ctrlHidden = true;
            }, 1000);
        },

        // 一直显示控制器
        showCtrlAlweys() {
            clearTimeout(hideCtrlTimer);
            this.ctrlHidden = false;
        },

        // 处理单击防抖
        handleClick() {
            clearTimeout(clickTimer);
            clickTimer = setTimeout(() => {
                this.changePlayState();
            }, 300);
        },

        // 双击清除防抖计时器并全屏
        handleDblClick() {
            clearTimeout(clickTimer);
            this.changeFullScreen();
        },

        // 改变视频播放状态
        changePlayState() {
            if (this.pause) {
                this.playVideo();
            } else {
                this.pauseVideo();
            }
        },

        // 播放
        playVideo() {
            this.pause = false;
            this.$refs.videoPlayer.play();
            speedInterval = setInterval(this.updateBufferSpeed, 1000);  // 启动定时更新缓冲网速任务
        },

        // 暂停
        pauseVideo() {
            this.pause = true;
            this.$refs.videoPlayer.pause();
            clearInterval(speedInterval);  // 清除定时任务
        },

        // 改变播放设置
        changeSetting(i) {
            if (i === 1) {
                if (this.setting.loop) {
                    this.setting.loop = false;
                } else {
                    this.setting.loop = true;
                }
            } else if (i === 2) {
                if (this.setting.autoplay) {
                    this.setting.autoplay = false;
                } else {
                    this.setting.autoplay = true;
                }
            } else if (i === 3) {
                if (this.setting.autonext) {
                    this.setting.autonext = false;
                } else {
                    this.setting.autonext = true;
                }
                this.$emit("update:autonext", this.setting.autonext);
            }
            const setting = JSON.stringify(this.setting);
            localStorage.setItem('playerSetting', setting);
        },

        // 点击小喇叭切换静音状态
        changeMutedState() {
            if (this.isMuted) {
                this.volume = localStorage.getItem('volume');
                this.isMuted = false;
            } else {
                this.volume = 0;
                this.isMuted = true;
            }
            this.$refs.videoPlayer.volume = this.volume / 100;
            this.volumeOrder = true;
        },

        // 改变音量
        changeVolume(per) {
            let volume = Math.floor(per * 100);
            this.volume = volume;
            this.$refs.videoPlayer.volume = volume / 100;
            if (this.isMuted && volume > 0) {
                this.isMuted = false;
            } else if (volume == 0 && !this.isMuted) {
                this.isMuted = true;
            }
            localStorage.setItem("volume", JSON.stringify(volume));
        },

        // 全屏/退出全屏
        changeFullScreen() {
            if (this.screenType !== 'full') {
                this.fullSrceen();
            } else {
                this.exitFullscreen();
            }
        },

        // 全屏
        fullSrceen() {
            this.screenType = 'full';
            const videoArea = document.querySelector('.player-video-area');
            if (videoArea.requestFullscreen) {
                videoArea.requestFullscreen();
            } else if (videoArea.mozRequestFullScreen) {
                videoArea.mozRequestFullScreen();
            } else if (videoArea.webkitRequestFullscreen) {
                videoArea.webkitRequestFullscreen();
            } else if (videoArea.msRequestFullscreen) {
                videoArea.msRequestFullscreen();
            }
            // 尝试强制横屏
            // if (screen.orientation) {
            //     screen.orientation.lock("landscape");
            // }
        },

        // 退出全屏
        exitFullscreen() {
            if (document.exitFullscreen) {
                document.exitFullscreen();
            } else if (document.mozCancelFullScreen) {
                document.mozCancelFullScreen();
            } else if (document.webkitExitFullscreen) {
                document.webkitExitFullscreen();
            } else if (document.msExitFullscreen) {
                document.msExitFullscreen();
            }
            this.screenType = 'normal';
        },

        // 处理全屏模式下按esc键触发的退出全屏事件
        handleVideoResize() {
            const videoArea = document.querySelector('.player-video-area');
            if (
                videoArea.clientWidth < window.innerWidth - 1 &&
                videoArea.clientHeight < window.innerHeight - 1 &&
                this.screenType !== 'normal'
            ) {
                this.screenType = 'normal'
            }
            if (videoArea.clientWidth > 980) {
                this.showDmInput = true;
            } else {
                this.showDmInput = false;
            }
        },

        // 拖动进度条的回调
        changeCurrentPer(curr) {
            this.currentPer = curr;
            // 计算当前时间
            let time = curr * this.duration;
            this.currentTime = time;
            this.$refs.videoPlayer.currentTime = time;
            this.initDanmuIndex(time);
        },

        // 实时更新当前播放的时间
        timeUpdate() {
            const time = this.$refs.videoPlayer.currentTime;
            // 如果是0就表示视频被循环播放了，需要初始化弹幕索引以及增加一个播放量
            if (time === 0) {
                // this.initDanmuIndex();
                this.increasePlay();
            }
            this.currentTime = time;
            // 计算进度条的位置
            this.currentPer = time / this.duration;
            // 展示该时间段的弹幕
            if (this.danmuOpen) {
                // this.displayDanmus(time);
            }
            this.lastTimePoint = time;
        },

        // 更新缓冲条
        updateBufferingBar() {
            const video = this.$refs.videoPlayer;
            const buffered = video.buffered;
            if (buffered.length > 0) {
                // 计算已缓冲的百分比并更新缓冲进度条
                this.bufferPer = buffered.end(buffered.length - 1) / this.duration;
            }
        },

        // 计算缓冲网速
        updateBufferSpeed() {
            const buffered = this.$refs.videoPlayer.buffered;
            if (buffered && buffered.length > 0) {
                let currByte = buffered.end(buffered.length - 1) / this.duration * this.videoSize;
                let differ = currByte - this.lastByte;    // 在1s内加载了这么多字节
                this.bufferSpeed = (differ / 1000).toFixed(2);
                this.lastByte = currByte;
            }
        },

        // 调整倍速
        changePlaybackRate(num) {
            if (num === 1.0) {
                this.playbackrate = '倍速';
            } else if (num === 2.0) {
                this.playbackrate = '2.0x';
            } else {
                this.playbackrate = `${num}x`;
            }
            this.$refs.videoPlayer.playbackRate = num;
        },

        // 鼠标悬停按钮时打开菜单
        enterBtn(name, index) {
            clearTimeout(menuHideTimer[index]);
            menuShowTimer[index] = setTimeout(() => {
                const obj = document.querySelector(`.${name}`);
                obj.classList.add('state-show');
            }, 200);
        },

        // 鼠标移出按钮时关闭菜单
        leaveBtn(name, index) {
            clearTimeout(menuShowTimer[index]);
            menuHideTimer[index] = setTimeout(() => {
                const obj = document.querySelector(`.${name}`);
                obj.classList.remove('state-show');
            }, 200);
        },

        // 格式化时间
        handleTime(time) {
            return handleTime(time);
        },

        // 视频播放结束
        ended() {
            if (this.setting.autonext) {
                this.$emit("next");
            }
        }
    },
    created() {
        // 同步音量
        if (localStorage.getItem("volume")) {
            this.volume = JSON.parse(localStorage.getItem("volume"));
        } else {
            localStorage.setItem("volume", JSON.stringify(this.volume));
        }
        // 同步播放设置
        if (localStorage.getItem("playerSetting")) {
            this.setting = JSON.parse(localStorage.getItem("playerSetting"));
        }
        // 判断是否允许自动播放
        if (!this.setting.autoplay) {
            this.pause = true;
        }
    },
    mounted() {
        this.changeWindowSize();
        window.addEventListener('resize', this.changeWindowSize);
        window.addEventListener('resize', this.handleVideoResize);
        document.addEventListener('keydown', (e) => this.handleKeyboard(e));
        // 监听 progress 事件，该事件在缓冲状态发生变化时触发
        // this.$refs.videoPlayer.addEventListener('progress', this.updateBufferingBar);
        // 默认播放音量不等于this.volume，所以挂载时更新同步一下音量
        this.$refs.videoPlayer.volume = this.volume / 100;
        setTimeout(() => {
            this.isMounted = true;
        }, 3000);
    },
    beforeUnmount() {
        window.removeEventListener('resize', this.changeWindowSize);
        document.removeEventListener('keydown', (e) => this.handleKeyboard(e));
        window.removeEventListener('resize', this.handleVideoResize);
    },
    watch: {
        "jumpTimePoint"(curr) {
            if (curr >= 0) {
                this.jumpTimePointAndPlay(curr);
                this.$emit("update:jumpTimePoint", -1);
            }
        },
        "autonext"(curr) {
            this.setting.autonext = curr;
            const setting = JSON.stringify(this.setting);
            localStorage.setItem('playerSetting', setting);
        },
        // 更换视频时初始化
        "videoUrl"() {
            this.canPlay = false;
            // this.initDanmuIndex(0);
        },
        // 监听登录状态，如果重新登录就要新增一个播放记录
        "$store.state.isLogin"(curr) {
            if (this.isMounted && curr) {
                this.increasePlay();
            }
        }
    }
}
</script>

<style scoped>
.player-wrap {
    position: relative;
}

.player-placeholder {
    position: absolute;
    display: flex;
    z-index: 0;
    width: 100%;
    height: 100%;
    flex-direction: column;
    flex-wrap: nowrap;
    box-shadow: 0 0 8px var(--bg3);
}

.player-placeholder-top {
    flex: 1;
    overflow: hidden;
    position: relative;
    background-color: #000;
}

.player-placeholder-bottom {
    position: relative;
    display: flex;
    height: 46px;
    align-items: center;
    padding: 0 12px;
    background: var(--bg1);
}

.player-placeholder-bottom-left {
    margin-right: 10px;
    width: 176px;
    height: 22px;
    background-color: rgba(0, 0, 0, 0.04);
}

.player-placeholder-bottom-right {
    height: 22px;
    flex-grow: 1;
    background-color: rgba(0, 0, 0, 0.04);
}

.player {
    position: relative;
    height: 100%;
    width: 100%;
}

.player-primary-area {
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-direction: column;
    flex-direction: column;
    -ms-flex-wrap: nowrap;
    flex-wrap: nowrap;
    height: 100%;
    width: 100%;
}

.player-video-area {
    -webkit-box-flex: 1;
    background-color: #000;
    -ms-flex: 1;
    flex: 1;
    overflow: hidden;
    position: relative;
}

.player-video-area[data-ctrl-hidden=true] {
    cursor: none;
}

.player-video-area * {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

.player-video-perch {
    -webkit-box-flex: 0;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex: none;
    flex: none;
    height: 100%;
    justify-content: center;
    position: relative;
    width: 100%;
}

.player-video-wrap {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    display: block;
    height: 100%;
    width: 100%;
}

.player-video-wrap video {
    content-visibility: visible;
    display: block;
    height: 100%;
    margin: auto;
    width: 100%;
}

.player-adv-dm-wrap, .player-bas-dm-wrap, .player-row-dm-wrap {
    cursor: pointer;
    height: 100%;
    left: 0;
    -webkit-mask-position: center;
    mask-position: center;
    overflow: hidden;
    pointer-events: none;
    position: absolute;
    top: 0;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    width: 100%;
    z-index: 2;
}

.player-row-dm-wrap {
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    justify-content: center;
    z-index: 11;
}

.state-paused .player-state {
    bottom: 62px;
    pointer-events: none;
    position: absolute;
    right: 34px;
    z-index: 48;
}

.state-paused[data-screen=full] .player-state, .state-paused[data-screen=web] .player-state {
    bottom: 107px;
}

.state-buff .player-state {
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-direction: column;
    flex-direction: column;
    height: 100%;
    justify-content: center;
    left: 0;
    position: absolute;
    top: 0;
    width: 100%;
    z-index: 48;
}

.player-state-buff-icon, .player-state-buff-text, .player-state-play {
    display: none;
}

.state-paused .player-state-play {
    background: url(~assets/img/play.svg) 50% no-repeat;
    display: block;
    height: 64px;
    width: 64px;
}

.state-buff .player-state-buff-icon {
    display: flex;
    height: 40px;
    width: 40px;
    align-items: center;
    justify-content: center;
}

.state-buff .player-state-buff-text {
    color: #fff;
    display: block;
    font-size: 14px;
    line-height: 1;
    margin-top: 8px;
    text-shadow: 0 1px 2px rgba(0,0,0,.7);
}

.player-loading-panel {
    background-color: #000;
    height: 100%;
    left: 0;
    position: absolute;
    top: 0;
    width: 100%;
    z-index: 57;
}

.player-masking {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 58;
}

.player-top-wrap {
    color: #fff;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    left: 0;
    pointer-events: none;
    position: absolute;
    top: 0;
    -webkit-transition: all .2s ease-in-out;
    transition: all .2s ease-in-out;
    width: 100%;
    z-index: 59;
}

.player-video-area[data-ctrl-hidden=true] .player-top-wrap {
    opacity: 0;
    visibility: hidden;
}

.player-video-area[data-ctrl-hidden=false] .player-top-wrap {
    opacity: 1;
    visibility: visible;
}

.player-top-left {
    left: 12px;
    margin-top: 18px;
    position: absolute;
}

.player-title {
    font-size: 20px;
    font-weight: 500;
    line-height: 24px;
    margin: 0 0 8px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    width: 100%!important;
    z-index: 2;
}

.player-video-area[data-screen=normal] .player-title {
    display: none;
}

.player-follow {
    -webkit-box-align: center;
    -ms-flex-align: center;
    -webkit-box-flex: 0;
    align-items: center;
    background-color: rgba(0,0,0,.4);
    border-radius: 26px;
    cursor: pointer;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex: 0;
    flex: 0;
    font-size: 12px;
    font-weight: 400;
    height: 32px;
    line-height: 32px;
    margin-bottom: 8px;
    pointer-events: all;
    text-align: center;
    width: 89px;
    z-index: 2;
}

.player-follow:hover {
    background-color: rgba(0,0,0,.6);
}

.player-follow-face {
    -webkit-box-flex: 0;
    border-radius: 50%;
    -ms-flex: none;
    flex: none;
    height: 24px;
    margin-left: 4px;
    vertical-align: bottom;
    width: 24px;
}

.player-follow-icon {
    -webkit-box-flex: 0;
    -ms-flex: none;
    flex: none;
    height: 12px;
    line-height: 12px;
    margin-left: 8px;
    width: 12px;
}

.player-follow-text {
    -webkit-box-flex: 0;
    -ms-flex: none;
    flex: none;
    margin-left: 5px;
}

.player-control-wrap {
    bottom: 0;
    left: 0;
    position: absolute;
    width: 100%;
    z-index: 75;
}

.player-control-mask {
    background: url(~assets/img/shadow.png) repeat-x bottom;
    bottom: 0;
    height: 100px;
    left: 0;
    pointer-events: none;
    position: absolute;
    -webkit-transition: opacity .2s ease-in-out;
    transition: opacity .2s ease-in-out;
    width: 100%;
    z-index: -1;
}

.player-control-top {
    bottom: 44px;
    left: 0;
    opacity: 1;
    position: absolute;
    right: 0;
    -webkit-transition: opacity .2s ease-out;
    transition: opacity .2s ease-out;
    visibility: visible;
}

.player-control-bottom {
    -webkit-box-pack: justify;
    -ms-flex-pack: justify;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    height: 35px;
    justify-content: space-between;
    line-height: 22px;
    margin: 30px 0 0;
    -webkit-transition: all .2s ease-out;
    transition: all .2s ease-out;
    width: 100%;
}

.player-control-bottom, .player-control-top {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    padding: 0 12px;
}

.player-ctrl-btn {
    fill: #fff;
    color: #fff;
    font-size: 0;
    height: 22px;
    line-height: 22px;
    outline: 0;
    position: relative;
    text-align: center;
    width: 36px;
    z-index: 2;
    opacity: 0.8;
}

.player-ctrl-btn:hover {
    opacity: 1;
}

.player-ctrl-btn-icon {
    cursor: pointer;
    width: 100%;
}

.common-svg-icon {
    display: -webkit-inline-box;
    display: -ms-inline-flexbox;
    display: inline-flex;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    height: 100%;
    width: 100%;
}

.player-ctrl-btn-icon .common-svg-icon {
    height: 22px;
    opacity: .9;
    vertical-align: middle;
}

.common-svg-icon svg {
    -webkit-transition: fill .15s ease-in-out;
    transition: fill .15s ease-in-out;
    height: 100% !important;
    width: 100% !important;
}

.player-control-bottom-left {
    -webkit-box-flex: 0;
    display: -webkit-inline-box;
    display: -ms-inline-flexbox;
    display: inline-flex;
    -ms-flex: none;
    flex: none;
}

.player-video-area[data-ctrl-hidden=false] .player-shadow-progress-area, .player-video-area[data-ctrl-hidden=true] .player-control-top {
    opacity: 0;
    visibility: hidden;
}

.player-video-area[data-ctrl-hidden=false] .player-control-top {
    opacity: 1;
    visibility: visible;
}

.player-video-area[data-ctrl-hidden=true] .player-control-bottom, .player-video-area[data-ctrl-hidden=true] .player-control-mask {
    opacity: 0;
}

.player-video-area[data-ctrl-hidden=false] .player-control-bottom, .player-video-area[data-ctrl-hidden=false] .player-control-mask {
    opacity: 1;
    -webkit-transition: opacity .2s ease-in;
    transition: opacity .2s ease-in;
}

.player-ctrl-time {
    font-size: 12px;
    margin-right: 10px;
    min-width: 90px;
}

.player-ctrl-time-label {
    height: 100%;
    position: absolute;
    text-align: center;
    white-space: nowrap;
    width: 100%;
}

.player-ctrl-time-divide {
    padding: 0 4px;
}

.player-control-bottom-right {
    -webkit-box-pack: end;
    -ms-flex-pack: end;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    justify-content: flex-end;
}

.player-video-area[data-screen=full] .player-control-bottom-right, .player-video-area[data-screen=web] .player-control-bottom-right {
    min-width: 378px;
}

.player-ctrl-quality {
    -webkit-box-flex: 0;
    -ms-flex: none;
    flex: none;
    font-size: 12px;
    margin-right: 10px;
    width: auto;
}

.player-ctrl-quality-result {
    cursor: pointer;
    font-size: 14px;
    font-weight: 600;
}

.player-ctrl-eplist {
    font-size: 14px;
    visibility: hidden;
    width: 0;
}

.player-video-area[data-screen=full] .player-ctrl-eplist, .player-video-area[data-screen=web] .player-ctrl-eplist {
    visibility: visible;
}

.player-ctrl-eplist-result {
    cursor: pointer;
    font-weight: 600;
    width: 100%;
}

.player-ctrl-playbackrate {
    font-size: 14px;
    width: 50px;
}

.player-ctrl-playbackrate-result {
    cursor: pointer;
    font-weight: 600;
    width: 100%;
}

.player-ctrl-playbackrate-menu {
    background-color: hsla(0,0%,8%,.9);
    border-radius: 2px;
    bottom: 41px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    display: none;
    left: 50%;
    margin: 0;
    padding: 0;
    position: absolute;
    text-align: center;
    -webkit-transform: translateX(-50%);
    transform: translateX(-50%);
    width: 70px;
}

.player-ctrl-playbackrate.state-show .player-ctrl-playbackrate-menu {
    display: block;
}

.player-ctrl-playbackrate-menu-item {
    cursor: pointer;
    height: 36px;
    line-height: 36px;
    position: relative;
}

.player-ctrl-playbackrate-menu-item:hover {
    background-color: hsla(0,0%,100%,.1);
}

.player-ctrl-playbackrate-menu-item.state-active {
    color: var(--brand_pink);
}

.player-ctrl-setting-box {
    bottom: 41px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    color: #fff;
    display: none;
    font-size: 12px;
    position: absolute;
    right: -48px;
    text-align: left;
}

.player-ctrl-setting.state-show .player-ctrl-setting-box {
    display: block;
}

.player-ctrl-setting-menu {
    border-radius: 2px !important;
}

.player-ctrl-setting-menu-left {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    height: 100%;
    padding: 6px 20px;
    width: 100%;
}

.player-ctrl-setting-loop, .player-ctrl-setting-autoplay, .player-ctrl-setting-autonext {
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    height: 32px;
    line-height: 32px;
    width: 100%;
}

.player-ctrl-volume-box {
    background: hsla(0,0%,8%,.9);
    border-radius: 2px;
    bottom: 41px;
    display: none;
    height: 100px;
    left: 50%;
    margin-left: -16px;
    position: absolute;
    width: 32px;
}

.player-ctrl-volume.state-show .player-ctrl-volume-box {
    display: block;
}

.player-ctrl-volume-number {
    color: #e5e9ef;
    font-size: 12px;
    height: 28px;
    line-height: 28px;
    margin-bottom: 2px;
    text-align: center;
    width: 100%;
}

.player-ctrl-volume-progress {
    height: 60px !important;
    margin: 0 auto;
}

.player-ctrl-volume-progress /deep/ .v-ui-area {
    -webkit-box-pack: center !important;
    -ms-flex-pack: center !important;
    justify-content: center !important;
}

.player-shadow-progress-area {
    bottom: 0;
    height: 2px;
    left: 0;
    position: absolute;
    right: 0;
    -webkit-transition: opacity .4s ease-in;
    transition: opacity .4s ease-in;
}

.player-video-area[data-ctrl-hidden=true] .player-shadow-progress-area {
    opacity: 1;
    visibility: visible;
}

.player-shadow-progress-schedule-wrap {
    height: 100%;
    position: relative;
    width: 100%;
}

.player-progress-schedule {
    background-color: hsla(0,0%,100%,.2);
    border-radius: 1.5px;
    bottom: 0;
    left: 0;
    overflow: hidden;
    position: absolute;
    right: 0;
    top: 0;
}

.player-progress-schedule-buffer, .player-progress-schedule-current {
    bottom: 0;
    left: 0;
    position: absolute;
    right: 0;
    top: 0;
    -webkit-transform: scaleX(0);
    transform: scaleX(0);
    -webkit-transform-origin: 0 0;
    transform-origin: 0 0;
}

.player-progress-schedule-buffer {
    background-color: hsla(0,0%,100%,.3);
}

.player-progress-schedule-current {
    background-color: var(--brand_pink);
}

.player-sending-bar {
    -ms-flex-negative: 0;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    height: 20px;
    padding: 0 12px;
    position: relative;
    -webkit-box-flex: 0;
    -webkit-box-pack: justify;
    -ms-flex-pack: justify;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    font-size: 13px;
    justify-content: space-between;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    width: 100%;
}

.player-sending-area {
    -webkit-box-flex: 0;
    -webkit-box-pack: justify;
    -ms-flex-pack: justify;
    background: #fff;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    align-items: center;
    -ms-flex: none;
    flex: none;
    font-size: 12px;
    height: 46px;
    justify-content: space-between;
    padding: 0 12px;
    font-size: 13px;
}

.player-video-info {
    -ms-flex-negative: 1;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    color: var(--text2);
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    flex-shrink: 1;
    height: 16px;
    line-height: 18px;
    margin-right: 24px;
    overflow: hidden;
    position: relative;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    white-space: nowrap;
}

.player-video-info-text {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.player-dm-root {
    -webkit-box-flex: 1;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex: auto;
    flex: auto;
    height: 34px;
}

.player-sending-bar .player-dm-outside {
    fill: hsla(0,0%,100%,.9);
}

.player-sending-area .player-dm-outside {
    fill: #61666D;
    transition: fill 0.1s;
}

.player-sending-area .player-dm-outside:hover {
    fill: var(--brand_pink);
}

.player-dm-switch, .player-dm-setting {
    cursor: pointer;
    height: 24px;
    line-height: 30px;
    margin-right: 12px;
    position: relative;
    width: 24px;
}

.player-dm-setting-wrap {
    display: none;
    background: none;
    border: none;
    border-radius: 4px 4px 0 0;
    bottom: 39px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    cursor: default;
    font-size: 12px;
    height: 359px;
    position: absolute;
    right: -149px;
    text-align: left;
    width: 320px;
    z-index: 1001;
}

.player-dm-setting.state-show .player-dm-setting-wrap {
    display: block;
}

.player-dm-setting-box {
    border-radius: 2px;
    bottom: 0;
    position: absolute;
    right: 0;
}

.player-dm-setting-left {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    height: 100%;
    padding: 12px 20px;
    width: 100%;
}

.player-dm-setting-left-opacity {
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    height: 16px;
    line-height: 16px;
    margin-bottom: 16px;
    width: 100%;
}

.player-dm-setting-left-speed {
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    height: 22px;
    line-height: 22px;
    width: 100%;
}

.dm-speed-items {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(1, 1fr);
    grid-gap: 10px;
    width: 100%;
}

.dm-speed-item {
    background: hsla(0,0%,100%,.2);
    border-radius: 2px;
    color: #fff;
    cursor: pointer;
    font-size: 12px;
    text-align: center;
}

.dm-speed-item.active {
    background: var(--brand_pink);
}

.player-dm-setting-left-title {
    color: #fff;
    height: 16px;
    line-height: 15px;
    text-align: left;
}

.player-dm-setting-left-content {
    -webkit-box-flex: 1;
    -ms-flex: 1;
    flex: 1;
    height: 12px;
    margin-left: 10px;
    width: 200px;
}

.player-dm-setting-left-progress {
    width: 182px !important;
    margin: auto;
}

.player-dm-setting-left-value {
    color: hsla(0,0%,100%,.8);
    cursor: default;
    font-size: 12px;
    line-height: 16px;
    text-align: right;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    width: 40px
}

.player-dm-inputbar {
    -ms-flex-align: center;
    -ms-flex-positive: 1;
    background: #F1F2F3;
    color: #999;
    flex-grow: 1;
    -webkit-box-align: center;
    -webkit-box-flex: 1;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    position: relative;
    border-radius: 8px;
    height: 32px;
    min-width: 300px;
    width: calc(100% - 72px);
}

.player-video-area[data-screen=full] .player-sending-bar .player-dm-inputbar, .player-video-area[data-screen=web] .player-sending-bar .player-dm-inputbar {
    background: none;
    background-color: hsla(0,0%,100%,.3);
    color: hsla(0,0%,100%,.6);
    margin: 0 auto;
    padding: 0;
}

.player-video-inputbar-wrap {
    -webkit-box-align: center;
    -webkit-box-flex: 1;
    align-items: center;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    position: relative;
    -ms-flex-align: center;
    border-radius: 8px 0 0 8px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    -ms-flex: 1;
    flex: 1;
    height: 100%;
    width: 200px;
}

.player-dm-btn-font {
    -webkit-box-flex: 0;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    cursor: pointer;
    -ms-flex: none;
    flex: none;
    position: relative;
    text-align: center;
    height: 30px;
    width: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    padding-left: 6px;
}

.player-dm-btn-font:hover .player-dm-btn-font-icon {
    fill: var(--brand_pink);
}

.player-mode-selection-container {
    background: hsla(0,0%,8%,.9);
    border-radius: 2px;
    bottom: 39px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    display: none;
    height: auto;
    left: 50%;
    margin-left: -108px;
    padding: 2px 0 0;
    position: absolute;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    width: 216px;
    z-index: 1001;
}

.player-dm-btn-font.state-show .player-mode-selection-container {
    display: block;
}

.player-mode-selection-panel {
    display: block;
    position: relative;
}

.player-mode-selection-row {
    font-size: 12px;
    line-height: 22px;
    margin: 10px 20px;
    min-height: 22px;
    position: relative;
    width: 176px;
}

.row-title {
    color: #fff;
    line-height: 16px;
    text-align: left;
}

.row-selection {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-wrap: wrap;
    flex-wrap: wrap;
    margin: 8px -8px 0 0;
}

.selection-span {
    background: hsla(0,0%,100%,.2);
    border-radius: 2px;
    color: #fff;
    cursor: pointer;
    font-size: 12px;
    margin-right: 8px;
    position: relative;
    text-align: center;
}

.selection-span:hover {
    background: hsla(0,0%,100%,.4);
}

.selection-span.type-fontsize {
    margin-bottom: 8px;
    width: 84px;
}

.selection-span.type-mode {
    fill: hsla(0,0%,100%,.8);
    background: transparent;
    color: hsla(0,0%,100%,.8);
    margin: -4px 22px 0 0;
}

.selection-span.type-mode:hover {
    fill: #fff;
    background: transparent;
    color: #fff;
}

.selection-span.active {
    background: var(--brand_pink);
    color: #fff;
}

.selection-span.active:hover {
    background: var(--brand_pink);
}

.selection-span.type-mode.active, .selection-span.type-mode.active:hover {
    fill: var(--brand_pink);
    background: transparent;
    color: var(--brand_pink)
}

.selection-icon {
    border-radius: 2px;
    display: block;
    font-size: 26px;
    height: 26px;
    overflow: hidden;
    position: relative;
    text-align: center;
    width: 26px;
    z-index: 1;
}

.player-dm-input {
    -webkit-box-flex: 1;
    -ms-flex-positive: 1;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    flex-grow: 1;
    height: 28px;
    line-height: 28px;
    min-width: 100px;
    padding: 0 10px 0 5px;
    width: 100%;
    z-index: 12;
    outline: 0;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    border: 0;
}

.player-video-area[data-screen=full] .player-sending-bar .player-dm-input, .player-video-area[data-screen=web] .player-sending-bar .player-dm-input {
    --el-text-color-placeholder: rgba(255,255,255,0.6);
}

.player-dm-input /deep/ .el-input__wrapper {
    padding: unset;
    box-shadow: none;
    width: 100%;
    background: none;
}

.player-dm-input /deep/ .el-input__inner {
    height: inherit;
    line-height: inherit;
    color: #18191C;
}

.player-video-area[data-screen=full] .player-sending-bar .player-dm-input /deep/ .el-input__inner, .player-video-area[data-screen=web] .player-sending-bar .player-dm-input /deep/ .el-input__inner {
    color: #fff;
}

.player-dm-btn-send {
    border-radius: 0 8px 8px 0;
    height: 100%;
    line-height: 32px;
    min-width: 62px;
    font-size: 13px;
    overflow: hidden;
    text-align: center;
    width: 62px;
    z-index: 13;
    background-color: var(--brand_pink);
    color: #fff;
    cursor: pointer;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    transition: 0.3s all;
}

.player-dm-btn-send:hover {
    background-color: var(--Pi4);
}

ol, ul {
    list-style: none;
    outline: none;
}

@media screen and (min-width: 750px) {
    .player-video-area[data-screen=full] .player-top-left, .player-video-area[data-screen=web] .player-top-left {
        left: 24px;
        margin-top: 24px;
    }

    .player-video-area[data-screen=full] .player-title, .player-video-area[data-screen=web] .player-title {
        margin-bottom: 12px;
    }

    .player-video-area[data-screen=full] .player-follow, .player-video-area[data-screen=web] .player-follow {
        height: 40px;
        line-height: 40px;
        margin-bottom: 12px;
        width: 97px;
    }

    .player-video-area[data-screen=full] .player-follow-face, .player-video-area[data-screen=web] .player-follow-face {
        height: 32px;
        width: 32px;
    }

    .player-video-area[data-screen=full] .player-control-entity, .player-video-area[data-screen=full] .player-control-wrap, .player-video-area[data-screen=web] .player-control-entity, .player-video-area[data-screen=web] .player-control-wrap {
        height: 93px;
        line-height: 93px;
    }

    .player-video-area[data-screen=full] .player-control-top, .player-video-area[data-screen=web] .player-control-top {
        bottom: 68px;
    }

    .player-video-area[data-screen=full] .player-control-bottom, .player-video-area[data-screen=web] .player-control-bottom {
        height: 45px;
        line-height: 34px;
        margin: 40px 0 0;
    }

    .player-video-area[data-screen=full] .player-control-bottom-left, .player-video-area[data-screen=web] .player-control-bottom-left {
        min-width: 316px;
    }

    .player-video-area[data-screen=full] .player-ctrl-btn, .player-video-area[data-screen=web] .player-ctrl-btn {
        height: 43px;
        line-height: 32px;
        width: 54px;
    }

    .player-video-area[data-screen=full] .player-ctrl-btn-icon .common-svg-icon, .player-video-area[data-screen=web] .player-ctrl-btn-icon .common-svg-icon {
        height: 28px;
    }

    .player-video-area[data-screen=full] .player-ctrl-time, .player-video-area[data-screen=web] .player-ctrl-time {
        height: 32px;
        width: 120px;
    }

    .player-video-area[data-screen=full] .player-ctrl-time-label, .player-video-area[data-screen=web] .player-ctrl-time-label {
        font-size: 14px;
        text-align: left;
        text-indent: 16px;
    }

    .player-video-area[data-screen=full] .player-control-bottom-center, .player-video-area[data-screen=web] .player-control-bottom-center {
        -webkit-box-flex: 1;
        -ms-flex-positive: 1;
        flex-grow: 1;
        height: 34px;
        line-height: 34px;
        padding: 0 60px;
    }

    .player-video-area[data-screen=full] .player-sending-bar, .player-video-area[data-screen=web] .player-sending-bar {
        height: 34px!important;
        line-height: 34px!important;
        margin: 0 auto;
        max-width: 729px;
        padding: 0;
    }

    .player-video-area[data-screen=full] .player-control-bottom-right, .player-video-area[data-screen=web] .player-control-bottom-right {
        min-width: 370px;
    }

    .player-video-area[data-screen=full] .player-ctrl-quality, .player-video-area[data-screen=web] .player-ctrl-quality {
        width: auto;
    }

    .player-video-area[data-screen=full] .player-ctrl-playbackrate, .player-video-area[data-screen=web] .player-ctrl-playbackrate {
        width: 66px;
    }

    .player-video-area[data-screen=full] .player-ctrl-quality-result, .player-video-area[data-screen=web] .player-ctrl-quality-result,
    .player-video-area[data-screen=full] .player-ctrl-eplist-result, .player-video-area[data-screen=web] .player-ctrl-eplist-result,
    .player-video-area[data-screen=full] .player-ctrl-playbackrate-result, .player-video-area[data-screen=web] .player-ctrl-playbackrate-result {
        font-size: 16px;
    }

    .player-video-area[data-screen=full] .player-ctrl-playbackrate-menu, .player-video-area[data-screen=web] .player-ctrl-playbackrate-menu,
    .player-video-area[data-screen=full] .player-ctrl-volume-box, .player-video-area[data-screen=web] .player-ctrl-volume-box {
        bottom: 74px;
    }

    .player-video-area[data-screen=full] .player-ctrl-setting-box, .player-video-area[data-screen=web] .player-ctrl-setting-box {
        bottom: 74px;
        right: -40px;
    }
}

@media screen and (max-width: 1200px) {
    .player-video-area[data-screen=full] .player-control-bottom-center, .player-video-area[data-screen=web] .player-control-bottom-center {
        padding: 0;
    }

    .player-video-area[data-screen=full] .player-control-bottom-right, .player-video-area[data-screen=web] .player-control-bottom-right {
        min-width: 272px;
    }
}

@media screen and (max-width: 1366px) {
    .player-video-area[data-screen=full] .player-control-bottom-left, .player-video-area[data-screen=web] .player-control-bottom-left {
        min-width: 273px;
    }
}

@media screen and (min-width: 1681px) {
    .player-placeholder-bottom {
        height: 56px;
    }

    .player-sending-area {
        font-size: 14px;
        height: 56px;
    }

    .player-video-info {
        line-height: 20px;
        margin-right: 36px;
    }

    .player-dm-switch, .player-dm-setting {
        margin-right: 18px;
    }

    .player-dm-inputbar {
        height: 40px;
        width: calc(100% - 84px);
    }

    .player-dm-btn-send {
        width: 64px;
        font-size: 14px;
        line-height: 40px;
    }
}
</style>
