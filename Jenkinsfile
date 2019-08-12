stage("Build and Publish") {
  def TASK = "d2l-tvm"
  node {
    ws("workspace/${TASK}") {
      checkout scm
      def ENV_NAME = "${TASK}-${EXECUTOR_NUMBER}";
      def EID = EXECUTOR_NUMBER.toInteger()
      def CUDA_VISIBLE_DEVICES=(EID*2).toString() + ',' + (EID*2+1).toString();

      sh label: "Build Environment", script: """set -ex
      rm -rf ~/miniconda3/envs/${ENV_NAME}
      conda create -n ${ENV_NAME} pip -y
      conda activate ${ENV_NAME}
      # pip install https://apache-mxnet.s3-us-west-2.amazonaws.com/dist/python/numpy/latest/mxnet_cu101mkl-1.5.0-py2.py3-none-manylinux1_x86_64.whl
      pip install git+https://github.com/d2l-ai/d2l-book
      python setup.py develop
      pip list
      """

      sh label: "Check Execution Output", script: """set -ex
      conda activate ${ENV_NAME}
      d2lbook build outputcheck
      """

      sh label: "Execute Notebooks", script: """set -ex
      conda activate ${ENV_NAME}
      export CUDA_VISIBLE_DEVICES=${CUDA_VISIBLE_DEVICES}
      export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64
      d2lbook build eval
      """

      sh label:"Build HTML", script:"""set -ex
      conda activate ${ENV_NAME}
      d2lbook build html
      """

      sh label:"Build PDF", script:"""set -ex
      conda activate ${ENV_NAME}
      which rsvg-convert
      rsvg-convert --version
      # d2lbook build pdf
      """

      sh label:"Build Package", script:"""set -ex
      conda activate ${ENV_NAME}
      """

      if (env.BRANCH_NAME == 'numpy2') {
        sh label:"Publish", script:"""set -ex
        conda activate ${ENV_NAME}
        d2lbook deploy html # pkg pdf
      """
      }
	}
  }
}