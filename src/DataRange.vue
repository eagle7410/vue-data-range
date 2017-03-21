<template>
	<div id="data-range" v-show="range.length">
		<ul class="labels" >
			<li
				v-for="(part, inx) in range"
				v-html="part.label"
				:data-num="inx"
				:class ='classLabel(part, inx)'
			></li>
		</ul>
		<div class="input" id="slider"></div>
		<div class="labelsYears" v-show="years.length">
			<span
				v-for="year in years"
				class="labelYear"
				:data-count="year.seasonsCount"
			>{{year.val}}</span>
		</div>
	</div>
</template>

<script>

export default {
	props : {
		dataRange : {
			type: Array,
			default: function () {
				return [];
 			}
		},
		widthLabel : {
			type: Number,
			default : 40
		},
		heightLabel : {
			type : Number,
			default : 2
		},
		typeTime : {
			type : String,
			default : 'seasons',
			validator : (val) => ~['seasons', 'years'].indexOf(val)
		}
	},
	data () {
		return {
			years : [],
			range : [],
			step : 1,
			min : 0,
			val : 0,
			max : 0
		}
	},
	methods : {
		/**
		 * Hook after change value.
		 */
		valChangeAfterHook() {
			let that = this;
			let w  = window;

			if (w.vueEvents && that.range.length) {
				w.vueEvents.$emit('dataRangeChangeValue',  {
					val  : that.val,
					data : that.range[that.val]
				});
			}

		},
		/**
		 * Get class for label.
		 */
		classLabel (data, num) {
			let that = this;
			let c = 'data-range-labels ' + that.typeTime;

			if (that.val == num) {
				c += ' active';
			}

			return c;
		},
		sliderIni () {
			let that = this;
			let widthDataRange = $('#data-range').width();
			let labelWidth = that.percent * widthDataRange / 100;
			let $labels = $('#data-range > .labels');

		 	that.setFirst();
			let $input = $('#data-range > .input');
            $input.find('.years').remove();
            $input.find('#first-point').remove();
			let $slider = $('#data-range > .input').slider({
				value : that.val,
				step  : that.step,
				min   : that.min,
				max   : that.max,
				slide : (ev, ui) =>  {

					if (!that.range[ui.value] || that.range[ui.value].isEmpty) {
						return false;
					}

					that.val = ui.value;
					that.valChangeAfterHook();
				}
			});

			let cls = 'years';
			let idPref = 'year_';

			$slider.append($('<div>', {
				id : 'first-point',
			}));

			let addYear = (isEmpty, num, isAfter) => {
				let call = (isAfter || false) ? 'after' :  'append';
				let setClass = cls;

				if (isEmpty) {
					setClass += ' empty'
				}

				$slider[call]($('<div>', {
					id : idPref + num,
					num : num,
					class : setClass,
					style : `width: ${that.percent}%;height: ${that.heightLabel}px;`
				}));
			}

			$slider.off('click');
			$(document).on('click', '.' + cls, (ev) => {
				let $el = $(ev.target);
				let num = $el.attr('num');

				if (num > -1 && !that.range[num].isEmpty) {
					that.val = num
					$slider.slider('value', num);
					that.valChangeAfterHook();
				}
			});

			if (that.range.length) {
				for (var i = 0; i< that.max ; ++i) {
					addYear(that.range[i].isEmpty, i);
				}
			}

			$labels.find('li').width(labelWidth);

			if (that.years.length) {
					let $years = $('#data-range > .labelsYears');
					let width = (100 / that.years.length) * widthDataRange / 100;
					$years.width(widthDataRange+(labelWidth*2));

					$years.find('.labelYear').each(function() {
						let $year = $(this);
						let count = $year.data('count');
						$year.width(count * labelWidth);
					});

			}
		},
		setFirst () {
			let that = this;
			let noEmpty = false;
			let len = that.range.length;
			for (var i = 0; i < len; ++i) {
				let p = that.range[i];

				if (!p.isEmpty) {
					noEmpty = i;
				}

				if (p.isFirst) {
					let isChange;

					if (p.isEmpty) {
						if (noEmpty !== false) {
							isChange = that.val != noEmpty;
							that.val = noEmpty;
							return isChange;
						} else {
							for (let k = (i+1); k<len; ++k) {
								p = that.range[k];
								if (!p.isEmpty) {
									isChange = that.val != k;
									that.val = k;
									return isChange;
								}
							}
						}
					}

					isChange = that.val != i;
					that.val = i;
					return isChange;
				}
			}

			return false;
		},
		/**
		 * property dataRange to range
		 */
		dataRangeToRange (range) {
			let that = this;
			let rangeLen = range.length;

			if (rangeLen) {
				that.range = range;

				let currentYear = new Date().getFullYear();
				let beCurrent = false;

				for (let i = 0; i < rangeLen; i++) {
					let range = that.range[i];
					let year = range.year;
					let isBe = false;

					if (year == 'all')
						continue;

					if (year == currentYear)
						beCurrent = true;

					that.years = that.years.map(el => {
						if(el.val == year) {
							el.seasonsCount++;
							isBe = true;
						}

						return el;
					});

					if (isBe) continue;

					that.years.push({
						val : year,
						seasonsCount : 1
					});
				}

				if(!beCurrent) {
					that.years.push({
						val : currentYear,
						seasonsCount : 1
					});
				}

			}

			that.val = 0;
			that.max = that.range.length - 1;

			if (that.max === -1) {
				that.percent = 0;
			} else {
				that.percent = 100 / that.max ;
			}
		}
	},
	created() {
		let that = this;
		let w = window;

		that.dataRangeToRange(that.dataRange)

		if (w.vueEvents) {
			w.vueEvents.$on('cartogramsDataRangeUpdate', function (ev) {
				that.typeTime = ev.typeTime;
				that.dataRangeToRange(ev.range);

				if (that.range[that.val].isEmpty) {
					that.setFirst()
					that.valChangeAfterHook();
				};

				$('#data-range > .input').slider('value', that.val);
				that.$forceUpdate();
				that.sliderIni();
			});
		}

	},
	mounted () {
		this.sliderIni();
	}
}
</script>
