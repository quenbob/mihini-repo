#*******************************************************************************
# Copyright (c) 2012 Sierra Wireless and others.
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Eclipse Public License v1.0
# which accompanies this distribution, and is available at
# http://www.eclipse.org/legal/epl-v10.html
#
# Contributors:
#     Sierra Wireless - initial API and implementation
#*******************************************************************************

ADD_CUSTOM_TARGET(doxygen_gen
    COMMAND cd ${CMAKE_CURRENT_SOURCE_DIR} && doxygen ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile
    COMMAND rm -rf ${CMAKE_CURRENT_BINARY_DIR}/C_User_API_doc
    COMMAND mv ${CMAKE_CURRENT_SOURCE_DIR}/C_User_API_doc ${CMAKE_CURRENT_BINARY_DIR}
)
