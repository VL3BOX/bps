<template>
    <AppLayout>
        <div class="v-raw" v-loading="loading">
            <el-tabs v-model="kungfuid" type="card">
                <el-tab-pane v-if="~~mountid" :label="$t('心法被动')" name="pasv" key="pasv"></el-tab-pane>
                <el-tab-pane
                    :label="showKungfuName(kungfu)"
                    :name="kungfu"
                    v-for="kungfu in kungfus"
                    :key="kungfu"
                ></el-tab-pane>
                <template v-if="~~mountid">
                    <el-tab-pane :label="$t('奇穴')" name="talent" key="talent" v-if="client === 'std'"></el-tab-pane>
                    <el-tab-pane :label="$t('镇派')" name="talent2" key="talent2" v-if="client === 'origin'"></el-tab-pane>
                    <el-tab-pane :label="$t('阵法')" name="zhenfa" key="zhenfa"></el-tab-pane>
                </template>
            </el-tabs>

            <!-- 搜索 -->
            <!-- <div class="m-raw-search m-archive-search">
            <el-input
                :placeholder="$t('搜索关键词')"
                v-model="search"
                class="input-with-select"
                @change="loadSkills"
            >
                <template slot="prepend">{{ $t('技能名') }}</template>
                <el-button slot="append" icon="el-icon-search"></el-button>
            </el-input>
        </div>-->

            <ul class="m-resource-list" v-if="data && data.length">
                <template v-if="kungfuid === 'talent'">
                    <el-collapse v-model="collapses">
                        <el-collapse-item
                            v-for="(item, index) in data"
                            :key="kungfuid + index"
                            :title="num2zh(index)"
                            :name="index"
                        >
                            <template v-for="(o, i) in item">
                                <li v-if="o" class="u-item" :key="i">
                                    <span class="u-id">ID:{{ o.SkillID }}</span>
                                    <img class="u-pic" :title="'IconID:' + o.IconID" :src="getIconURL(o.IconID)" />
                                    <div class="u-primary">
                                        <a class="u-name" :href="o.SkillID | skillLink" target="_blank">
                                            {{ o.Name }}
                                            <em v-if="o.SkillName">({{ o.SkillName }})</em>
                                        </a>
                                        <span class="u-content">{{ o.Desc | filterRaw }}</span>
                                        <div class="u-remarks">
                                            <span class="u-remark">Level : {{ o.Level }}</span>
                                            <span class="u-remark">Remark : {{ o.Remark }}</span>
                                            <span v-if="o.HelpDesc" class="u-remark">HelpDesc : {{ o.HelpDesc }}</span>
                                            <span v-if="o.SimpleDesc" class="u-remark"
                                                >SimpleDesc : {{ o.SimpleDesc }}</span
                                            >
                                            <span v-if="o.SpecialDesc" class="u-remark"
                                                >SpecialDesc : {{ o.SpecialDesc }}</span
                                            >
                                        </div>
                                    </div>

                                    <!-- <skill-wiki
                                        :wiki="wikis[o.SkillID]"
                                        :key="kungfuid + o.SkillID"
                                        :sourceId="o.SkillID"
                                    /> -->
                                </li>
                            </template>
                        </el-collapse-item>
                    </el-collapse>
                </template>
                <template v-else>
                    <li v-for="(o, i) in data" class="u-item" :key="i">
                        <span class="u-id" v-if="o">ID:{{ o.SkillID }}</span>
                        <img class="u-pic" :title="'IconID:' + o.IconID" :src="getIconURL(o.IconID)" />
                        <div class="u-primary">
                            <a class="u-name" :href="o.SkillID | skillLink" target="_blank">
                                {{ o.Name }}
                                <em v-if="o.SkillName">({{ o.SkillName }})</em>
                            </a>
                            <span class="u-content">{{ o.Desc | filterRaw }}</span>
                            <div class="u-remarks">
                                <span class="u-remark">Level : {{ o.Level }}</span>
                                <span class="u-remark">Remark : {{ o.Remark }}</span>
                                <span v-if="o.HelpDesc" class="u-remark">HelpDesc : {{ o.HelpDesc }}</span>
                                <span v-if="o.SimpleDesc" class="u-remark">SimpleDesc : {{ o.SimpleDesc }}</span>
                                <span v-if="o.SpecialDesc" class="u-remark">SpecialDesc : {{ o.SpecialDesc }}</span>
                            </div>
                        </div>

                        <!-- <skill-wiki
                            v-if="o"
                            :wiki="wikis[o.SkillID]"
                            :key="kungfuid + o.SkillID + o.Level"
                            :sourceId="o.SkillID"
                        /> -->
                    </li>
                </template>
            </ul>
            <el-alert v-else class="m-archive-null" :title="$t('没有找到相关条目')" type="info" center show-icon></el-alert>
        </div>
    </AppLayout>
</template>

<script>
import AppLayout from "@/layout/AppLayout.vue";
import xfmap from "@jx3box/jx3box-data/data/xf/xf.json";
import { getSkills, getTalents, getTalents2 } from "../service/raw";
import { __iconPath, __ossRoot } from "@jx3box/jx3box-common/data/jx3box.json";
import cloneDeep from "lodash/cloneDeep";
import { getLink } from "@jx3box/jx3box-common/js/utils";
import kungfumap_std from "@/assets/data/kungfu_std.json";
import kungfumap_origin from "@/assets/data/kungfu_origin.json";
import pasvmap from "@/assets/data/pasv.json";
import zhenfamap from "@/assets/data/zhenfa.json";
import kungfus from "@/assets/data/kungfuid.json";
import kungfus_origin from "@/assets/data/kungfuid_origin.json";
import { getSkillWiki } from "@/service/helper";
import { flattenDeep } from "lodash";

// components
// import skillWiki from "@/components/skill/skill_wiki.vue";
export default {
    name: "Raw",
    components: {
        // skillWiki,
        AppLayout,
    },
    props: [],
    data: function () {
        return {
            data: [],
            loading: false,
            // 默认展开全部
            collapses: Array.from({ length: 12 }, (_, k) => k),

            kungfuid: "pasv",
            search: "",

            wikis: {},
            talents: null,
            talents2: null,
        };
    },
    computed: {
        subtype: function () {
            return this.$route.query.subtype || "通用";
        },
        mountid: function () {
            return xfmap[this.subtype]?.["id"] || "0";
        },
        kungfus: function () {
            return this.kungfumap[this.mountid]["kungfus"];
        },
        skill_ids: function () {
            return this.kungfumap[this.mountid]["skills"][this.kungfuid];
        },
        pasv_skills: function () {
            return pasvmap[this.subtype][this.client];
        },
        zhenfa_skills: function () {
            return (this.mountid && zhenfamap[this.mountid]) || [];
        },
        talent_skills: function () {
            return (this.mountid && this.talents?.[this.mountid]) || [];
        },
        talent2_skills: function () {
            return (this.mountid && this.talents2?.[this.mountid]) || [];
        },
        ids: function () {
            let skills = {
                pasv: this.pasv_skills,
                zhenfa: this.zhenfa_skills,
                talent: this.talent_skills.flat(),
                talent2: this.talent2_skills.flat(),
            };
            return skills[this.kungfuid] ? skills[this.kungfuid]?.join(",") : this.skill_ids?.join(",");
        },
        client: function () {
            return this.$store.state.client || "std";
        },
        params: function () {
            return {
                ids: this.ids,
                client: this.client,
            };
        },
        kungfumap: function () {
            return this.client == "origin" ? kungfumap_origin : kungfumap_std;
        },
    },
    methods: {
        loadSkills: function () {
            this.loading = true;
            getSkills(this.params)
                .then((res) => {
                    let data = res.data || [];
                    if (this.kungfuid == "zhenfa") {
                        this.data = data;
                    } else if (this.kungfuid === "talent") {
                        this.data = this.handleTalentData(data);
                    } else {
                        this.data = this.removeLowLevelSkills(data);
                    }
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        removeLowLevelSkills: function (data) {
            let arr = [];
            let _data = [];
            // 只保留最高等级
            data.forEach((item) => {
                if (!arr.includes(item.SkillID)) {
                    arr.push(item.SkillID);
                    _data.push(item);
                } else {
                    let i = arr.indexOf(item.SkillID);
                    if (~~item.Level > ~~_data[i].Level) {
                        _data[i] = item;
                    }
                }
            });
            return _data;
        },
        handleTalentData: function (data) {
            const skills = {
                talent: this.talent_skills,
                // talent2: this.talent2_skills
            };
            const talentSkills = cloneDeep(skills[this.kungfuid]);
            return talentSkills.map((item) => {
                return item.map((SkillID) => {
                    const currentTalent = data.find((d) => d.SkillID == SkillID);

                    return currentTalent;
                });
            });
        },
        num2zh: function (num) {
            const zh = ["一", "二", "三", "四", "五", "六", "七", "八", "九", "十", "十一", "十二"];
            return `第${zh[num]}重`;
        },
        loadWiki: function (skills) {
            getSkillWiki("skill", { source_id: skills, client: this.client }).then((res) => {
                if (!Array.isArray(res.data.data)) {
                    // 后端为空返回空数组，右值返回对象
                    this.wikis = res.data.data;
                }
            });
        },
        showKungfuName: function (val) {
            const kungfuMap = this.client === "origin" ? kungfus_origin : kungfus;
            return kungfuMap[val];
        },
        //区分包括不限于技能大全的icon所属client
        getIconURL:function(iconID){
            const iconURL = this.client ==="origin" ? __iconPath + "origin_icon/" + iconID + ".png" : __iconPath + "icon/" + iconID + ".png";
            return iconURL;
        }
    },
    filters: {
        filterRaw: function (str) {
            str = str && str.replace(/\\n/g, "\n");
            str = str && str.replace(/(\<TALENT.*?\>)/g, "\n$1");
            str = str && str.replace(/(\<EnchantID.*?\>)/g, "\n$1");
            return str;
        },
        //iconURL: function (id) {
        //    return __iconPath + "icon/" + id + ".png";
        //},

        skillLink: function (id) {
            return getLink("skill", id);
        },

    },
    watch: {
        subtype: {
            // immediate: true,
            handler: function () {
                // this.kungfuid = this.kungfus[0];
                this.kungfuid = "pasv";
                this.loadSkills();
            },
        },
        kungfuid: {
            immediate: true,
            handler: function (val) {
                this.loadSkills();
            },
        },
        // data: {
        //     deep: true,
        //     handler(val) {
        //         const _skillIds = flattenDeep(val).map((item) => item.SkillID);
        //         const skills = _skillIds.join(",");
        //         this.loadWiki(skills);
        //     },
        // },
    },
    mounted: async function () {
        this.talents = await getTalents();
        this.talents2 = await getTalents2();
    },
};
</script>

<style lang="less">
@import "../assets/css/raw.less";
</style>
