<template>
	<div class="edit" v-if="operatorBox">
		<el-row>
			<el-col :span="12">
				<div v-if="operator" class="operator-img" :class="operator.key"></div>
			</el-col>
			<el-col :span="12">
				<el-form size="mini">
					<el-form-item>
						<el-button-group>
							<el-button
								size="mini"
								round
								:class="operatorBox.eliteLevel === 0 ? 'elite-selected' : ''"
								@click="changEliteLevel(0)"
								:disabled="!operator"
							>
								无精
							</el-button>
							<el-button
								size="mini"
								round
								:disabled="!operator || operator.rate <= 2"
								:class="operatorBox.eliteLevel === 1 ? 'elite-selected' : ''"
								@click="changEliteLevel(1)"
							>
								精一
							</el-button>
							<el-button
								size="mini"
								round
								:disabled="!operator || operator.rate <= 3"
								:class="operatorBox.eliteLevel === 2 ? 'elite-selected' : ''"
								@click="changEliteLevel(2)"
							>
								精二
							</el-button>
						</el-button-group>
					</el-form-item>
					<el-form-item>
						等级:
						<el-input-number
							v-model="operatorBox.expLevel"
							:min="1"
							:max="expMaxLevel"
							label="等级"
							size="mini"
						/>
					</el-form-item>
					<el-form-item>
						潜能:
						<el-input-number
							v-model="operatorBox.potential"
							:min="0"
							:max="MAX_PROTENTIAL_LEVEL"
							label="潜能"
							size="mini"
						/>
					</el-form-item>
				</el-form>
			</el-col>
		</el-row>
		<el-row>
			<el-col>
				<el-slider
					v-model="operatorBox.skillsLevel"
					:min="1"
					:max="7"
					:step="1"
					:marks="{ 1: '1', 7: '7', 4: '技能等级' }"
					show-stops
					@input="changeSkillLevel"
				>
				</el-slider>
			</el-col>
		</el-row>
		<el-row>
			<el-col class="skill-group">
				<skill
					v-if="operatorBox.skills"
					v-for="count in operatorBox.skills.length"
					:key="count"
					:skill-key="operator.skills[count - 1].k"
					:skill-name="operator.skills[count - 1].n"
					:elite-level="operatorBox.eliteLevel"
					:skill-level="operatorBox.skills[count - 1]"
					@skill-up="operatorBox.skills[count - 1]++"
					@skill-down="operatorBox.skills[count - 1]--"
				/>
			</el-col>
		</el-row>
		<el-divider />
		<el-row>
			<h3>使用练度模板:</h3>
		</el-row>
		<el-row class="level-button-box">
			<div class="level-button-group">
				<el-button size="small" @click="setOperatorLevel(0, -1, 4)">
					精零满级4级技能
				</el-button>
				<el-button size="small" @click="setOperatorLevel(1, -1, 7)">
					精一满级7级技能
				</el-button>
				<el-button size="small" @click="setOperatorLevel(2, 30, 7)">
					精二30级7级技能
				</el-button>
				<el-button size="small" @click="setOperatorLevel(2, 70, 7)">
					精二70级7级技能
				</el-button>
				<el-button size="small" @click="setOperatorLevel(2, -1, 7)">
					精二满级7级技能
				</el-button>
			</div>
			<el-divider direction="vertical" />
			<div class="level-button-group">
				<el-button size="small" disabled @click="setOperatorLevel(0, 0, 0)">
					全站平均练度
				</el-button>
				<el-button size="small" @click="setOperatorLevel(2, -1, 10)"> 满练 </el-button>
				<el-button size="small"> 自定义1 </el-button>
				<el-button size="small"> 自定义2 </el-button>
				<el-button size="small"> 自定义3 </el-button>
			</div>
			<el-divider direction="vertical" />
			<div class="level-button-group">
				<el-button size="small" @click="setOperatorLevel(0, 0, 1, 0)"> 重置 </el-button>
				<el-button size="small" @click="setOperatorLevel(2, -1, 10, MAX_PROTENTIAL_LEVEL)">
					满配满练
				</el-button>
				<el-button size="small" disabled> 编辑 </el-button>
				<el-button size="small" disabled> 编辑 </el-button>
				<el-button size="small" disabled> 编辑 </el-button>
			</div>
		</el-row>
		<el-divider />
		<el-row>
			<el-col class="action-group">
				<el-button size="small"> 删除选中 </el-button>
			</el-col>
		</el-row>
	</div>
</template>

<script lang="ts" setup>
import type { IOperatorLight, IOperatorBox } from '@/global/entity/operator'
import { getExpMaxLevel } from '@/utils/operator'
import { useStore } from 'vuex'
import Skill from '@/components/Skill.vue'

const MAX_PROTENTIAL_LEVEL = 5
const MAX_NORAML_SKILL_LEVEL = 7

const store = useStore()
const props = defineProps({
	operatorId: Number
})

const changeSkillLevel = (val: number) => {
	for (let i = 0; i < operatorBox.skills.length; ++i)
		if (!(val === 7 && operatorBox.skills[i] >= 7))
			// ! 特殊情况(7级技能+专精)
			operatorBox.skills[i] = val
}

let operatorBox = $computed(() => {
	let operatorBoxes: IOperatorBox[] = store.state.userBox.box
	let operatorBox: IOperatorBox | undefined
	operatorBoxes.forEach((box) => {
		if (props.operatorId === box.operatorId) operatorBox = box
	})
	return operatorBox
})

let operator = $computed(() => {
	let operatorId = props.operatorId ? props.operatorId : 0
	let operator: IOperatorLight = store.state.operators.operatorsLight[operatorId - 1]
	return operator
})

const changEliteLevel = (level: number) => {
	if (operatorBox) operatorBox.eliteLevel = level
}
let expMaxLevel = $computed(() =>
	getExpMaxLevel(operatorBox ? operatorBox.eliteLevel : 0, operator ? operator.rate : 0)
)

const setOperatorLevel = (elite: number, exp: number, skill: number, potential: number = -1) => {
	operatorBox.eliteLevel = elite
	if (exp === -1) {
		// * max exp level
		operatorBox.expLevel = expMaxLevel
	} else {
		operatorBox.expLevel = exp
	}
	operatorBox.skillsLevel = skill > MAX_NORAML_SKILL_LEVEL ? MAX_NORAML_SKILL_LEVEL : skill
	setTimeout(() => {
		for (let i = 0; i < operatorBox.skills.length; ++i) operatorBox.skills[i] = skill
	}, 20)

	operatorBox.potential = potential === -1 ? operatorBox.potential : potential
}
</script>

<style lang="scss" scoped>
@import '@/assets/OperatorImage/operatorImgOrigin.scss';

.operator-img {
	border: 1px solid black;
	@include operatorOrigin;
	zoom: 1.3;
}
.operator-choose {
	width: 130px;
	height: 130px;
	border: 1px solid black;
	line-height: 130px;
	text-align: center;
}
.level-button-box {
	display: flex;
	flex-direction: row;
	.level-button-group {
		display: flex;
		flex-direction: column;
		.el-button {
			margin: 0;
			margin-bottom: 10px;
		}
	}
}
.elite-selected {
	background-color: #409eff;
	color: white;
}
.action-group {
	display: flex;
	justify-content: space-around;
	.el-button {
		width: 30%;
		min-width: 35px;
	}
}
.skill-group {
	display: flex;
	margin-top: 15px;
	justify-content: space-between;
}
</style>
