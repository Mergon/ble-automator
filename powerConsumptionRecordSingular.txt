#!/usr/bin/env python

__author__ = ‘Merijn van Tooren’

import math
from bleAutomator2 import *
import ConfigUtils
from ConversionUtils import *
from Bluenet import *

if __name__ == ‘__main__’:
	try:
		parser = optparse.OptionParser(usage='%prog [-v] [-i <interface>] [-f <output file>] [-c <config file>] \n\nExample:\n\t%prog -i hci0 -f data.txt -c config.json',
									version='0.1')
		
		parser.add_option('-i', '--interface',
				action='store',
				dest="interface",
				type="string",
				default="hci0",
				help='HCI interface to be used.'
				)
		parser.add_option('-f', '--file',
				action='store',
				dest="data_file",
				type="string",
				default="data.txt",
				help='File to store the data'
				)
		parser.add_option('-v', '--verbose',
				action='store_true',
				dest="verbose",
				help='Be verbose.'
				)
		parser.add_option('-c', '--config',
				action='store',
				dest="configFile",
				default="config.json",
				help='Config file (json)'
				)
		
		options, args = parser.parse_args()
		
	except Exception, e:
		print e
		print “For help use —help”
		sys.exit(2)
		
	ble = BleAutomator(options.interface, options.verbose)
	
	addresses = ConfigUtils.readAddresses(options.configFile)
	if (not addresses):
		sys.exit(1)
	address_ind = 0
	
	if (len(addresses) == 1):
        	cycle = 0
        	ble.connect(addresses[0])
	else:
		cycle = 1
	
	# Endless loop:
	while (True):
		# Connect to peer device.
		if (cycle == 1):
			ble.connect(addresses[address_ind])
		
		# Make the crownstone sample the power, give it some time to sample
		ble.writeCharacteristic(CHAR_SAMPLE_POWER, [3])
		time.sleep(0.5)
		
		# Read the power consumption
		power = ble.readCharacteristic(CHAR_READ_POWER_CONSUMPTION)