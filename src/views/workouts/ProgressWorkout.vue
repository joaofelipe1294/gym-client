<template>
  <div id="workout-progress-view">
    <UserNavbar /> 
    <h3 class="has-text-centered is-size-4" style="margin-top: 3%">
      Treino {{ workout_report.workout.name }}
    </h3>

    <div v-show="is_every_exercise_complete">
      <button @click="finish_workout" id="btn-finish-workout" class="button is-danger is-outlined" style="margin-left: 10%; margin-right: 10%; width: 80%; margin-top: 5%;">
        Concluir treino
      </button>
    </div>

    <hr>

    <div v-for="exercise_report in workout_report.exercise_reports" :key="exercise_report.id" class="card workout-exercise-card">
      <header class="card-header" :class="card_header_background_color(exercise_report)">
        <p class="card-header-title has-text-white">{{ exercise_report.workout_exercise.exercise.name }}</p>
      </header>
      <div class="card-content" v-show="is_exercise_complete(exercise_report)">
        <p>Séries: {{ exercise_report.workout_exercise.series_number }}</p>
        <p>Repetições: {{ exercise_report.workout_exercise.repetitions }}</p>
        <p>Tempo de descanço: {{exercise_report.workout_exercise.rest_time }}</p>
        <button id="btn-start-exercise" @click.prevent="redirect_to_new_exercise_report(exercise_report.id)" class="button is-info is-fullwidth is-outlined" style="margin-top: 5%;">
          Iniciar
        </button>
      </div>
    </div>

  </div>
</template>

<script>
import UserNavbar from '@/components/navbars/UserNavbar'
import WorkoutReportService from '../../services/WorkoutReportService'
import router from '@/router/index'
export default {
  data(){
    return {
      workout_report: {
        workout: {
          name: ''
        }
      },
      is_every_exercise_complete: false,
    }
  },
  components: {
    UserNavbar,
  },
  created(){
    WorkoutReportService.progress().then(response => {      
      this.workout_report = response.data.workout_report
      this.is_every_exercise_complete = this.workout_report.exercise_reports.every(exercise_report => {
        return exercise_report.status == 'complete'
      })
    }).catch(error => console.log(error))
  },
  methods: {
    redirect_to_new_exercise_report(exercise_report_id){
      router.push(`/user/exercise_report/${exercise_report_id}`)
    },
    is_exercise_complete(exercise_report){
      if (exercise_report.status == 'complete')
        return false
      else
        return true
    },
    card_header_background_color(exercise_report){
      if(exercise_report.status == 'complete')
        return 'has-background-info'
      else
        return 'has-background-primary'
    },
    finish_workout(){
      const workout_report = { 
        id: this.$route.params.id, 
        status: 'complete'  
      }
      WorkoutReportService.update(workout_report).then(() => {
        router.push("/home/user")
      }).catch(error => console.log(error))
    }
  }
}
</script>