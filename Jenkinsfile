#!/usr/bin/groovy

/*
 * Copyright (C) 2016-2017 Pelagicore AB
 *
 * Permission to use, copy, modify, and/or distribute this software for
 * any purpose with or without fee is hereby granted, provided that the
 * above copyright notice and this permission notice appear in all copies.
 *
 * THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL
 * WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED
 * WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR
 * BE LIABLE FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES
 * OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS,
 * WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION,
 * ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS
 * SOFTWARE.
 *
 * For further information see LICENSE
 */

// Runs a shell command in the vagrant vm
def runInVagrant = { String command ->
    sh "cd ${env.WORKSPACE} && vagrant ssh -c 'cd dbus-proxy && ${command}'"
}

def shutdownVagrant = {
    // In case the destroy step fails (happens sometimes with hung up ssh connections)
    // we retry a couple of times to make sure it shuts down and is destroyed.
    retry(5) {
        sh "cd ${env.WORKSPACE} && vagrant destroy -f"
    }
}

node {
    try {

        // Stages are subtasks that will be shown as subsections of the finiished build in Jenkins.
        stage('Download') {
            // Delete old files
            sh 'rm -rf .[^.] .??* *'
            // Checkout the git repository and refspec pointed to by jenkins
            checkout scm
            // Update the submodules in the repository.
            sh 'git submodule update --init'
        }

        stage('StartVM') {

            // Start the machine (destroy it if present) and provision it
            shutdownVagrant()
            withEnv(["APT_CACHE_SERVER=10.8.36.16"]) {
                sh "cd ${env.WORKSPACE} && vagrant up"
            }
        }

        // Configure the software with cmake
        stage('Configure') {
            // Remove old build directory
            runInVagrant("rm -rf build")
            // Run cmake
            runInVagrant("cmake -H. -Bbuild")
        }

        // Build the rest of the projekt
        stage('Build') {
            runInVagrant("cd build && make")
        }

        stage('Install') {
            runInVagrant("cd build && sudo make install")
        }

        stage('ComponentTest') {
            runInVagrant("cd component-test && py.test -v --junitxml=component-test-results.xml")
        }

        stage('Clang') {
            // Most build parameters default to off, but not tests. We don't want static
            // analysis for our tests.
            runInVagrant("sh ./vagrant-cookbook/build/clang-code-analysis.sh . clang")
        }


        stage('Artifacts') {
            // Store the artifacts of the entire build
            archive "**/*"

            publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'clang/scan_build_output/report',
                reportFiles: 'index.html',
                reportName: 'Clang report'
            ])

            // Store the component test results and graph them
            step([$class: 'JUnitResultArchiver', testResults: '**/component-test-results.xml'])
        }
    }

    catch(err) {
        // Do not add a stage here.
        // When "stage" commands are run in a different order than the previous run
        // the history is hidden since the rendering plugin assumes that the system has changed and
        // that the old runs are irrelevant. As such adding a stage at this point will trigger a
        // "change of the system" each time a run fails.
        println "Something went wrong!"
        currentBuild.result = "FAILURE"
    }

    finally {
        // Always try to shut down the machine
        shutdownVagrant()
    }
}

