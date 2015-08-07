#Protocol that a DFS datanode uses to communicate with the NameNode

## Introduction ##

Protocol that a DFS datanode uses to communicate with the NameNode.
It's used to upload current load information and block reports.
The only way a NameNode can communicate with a DataNode is by
returning values from these functions.

I will only list the important fields and methods at here, the detail
info please reference the source code in "org/apache/hadoop/hdfs/server/protocol/DatanodeProtocol.java"

## Details ##

```
	/**
	 * Determines actions that data node should perform when receiving a
	 * datanode command.
	 */
	final static int DNA_UNKNOWN = 0; // unknown action
	final static int DNA_TRANSFER = 1; // transfer blocks to another datanode
	final static int DNA_INVALIDATE = 2; // invalidate blocks
	final static int DNA_SHUTDOWN = 3; // shutdown node
	final static int DNA_REGISTER = 4; // re-register
	final static int DNA_FINALIZE = 5; // finalize previous upgrade
	final static int DNA_RECOVERBLOCK = 6; // request a block recovery
	final static int DNA_ACCESSKEYUPDATE = 7; // update access key
```


```
	/**
	 * Register Datanode.
	 * 
	 * @see org.apache.hadoop.hdfs.server.datanode.DataNode#dnRegistration
	 * @see org.apache.hadoop.hdfs.server.namenode.FSNamesystem#registerDatanode(DatanodeRegistration)
	 * 
	 * @return updated
	 *         {@link org.apache.hadoop.hdfs.server.protocol.DatanodeRegistration}
	 *         , which contains new storageID if the datanode did not have one
	 *         and registration ID for further communication.
	 */
	public DatanodeRegistration registerDatanode(
			DatanodeRegistration registration) throws IOException;

	/**
	 * sendHeartbeat() tells the NameNode that the DataNode is still alive and
	 * well. Includes some status info, too. It also gives the NameNode a chance
	 * to return an array of "DatanodeCommand" objects. A DatanodeCommand tells
	 * the DataNode to invalidate local block(s), or to copy them to other
	 * DataNodes, etc.
	 */
	@Nullable
	public DatanodeCommand[] sendHeartbeat(DatanodeRegistration registration,
			long capacity, long dfsUsed, long remaining, int xmitsInProgress,
			int xceiverCount) throws IOException;

	/**
	 * blockReport() tells the NameNode about all the locally-stored blocks. The
	 * NameNode returns an array of Blocks that have become obsolete and should
	 * be deleted. This function is meant to upload *all* the locally-stored
	 * blocks. It's invoked upon startup and then infrequently afterwards.
	 * 
	 * @param registration
	 * @param blocks
	 *            - the block list as an array of longs. Each block is
	 *            represented as 2 longs. This is done instead of Block[] to
	 *            reduce memory used by block reports.
	 * 
	 * @return - the next command for DN to process.
	 * @throws IOException
	 */
	public DatanodeCommand blockReport(DatanodeRegistration registration,
			long[] blocks) throws IOException;

	/**
	 * blockReceived() allows the DataNode to tell the NameNode about
	 * recently-received block data, with a hint for pereferred replica to be
	 * deleted when there is any excessive blocks. For example, whenever client
	 * code writes a new Block here, or another DataNode copies a Block to this
	 * DataNode, it will call blockReceived().
	 */
	public void blockReceived(DatanodeRegistration registration,
			Block blocks[], String[] delHints) throws IOException;
```